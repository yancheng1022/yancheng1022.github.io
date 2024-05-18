---
title: PBRTQC
date: 2024/01/08
categories:
  - coding
tags:
  - PBRTQC
  - 技术文章
abbrlink: 22105
---
# 1、基本概念

## 1.1、PBRTQC

基于患者样本的实时质量控制（patient-based real-time quality control，PBRTQC）系统是一种基于统计学以及数学模型的质量控制方法，是根据患者样本检测结果，利用统计学模型建立的一套以实时监测实验室检测质量的模型或者规则，是医学检验实验室提高质量控制体系的重要发展方向，最早由Hoffmann和Waid于1965年提出

## 1.2、PBRTQC模型

1. 五个关键参数：上截断值（UTL）、下截断值（LTL）、浮动窗口大小（N）、上控制限（UCL）、下控制限（LCL）
2. 排除规则：对于偏态分布以及存在极端值的分布数据，应选择一个适宜的截断值区间，将在截断值区间外的数据进行缩尾或者去除
3. SPC算法：浮动均值（MA）、浮动分位数（MQ）、指数加权移动平均值（EWMA）用于监测定值误差（CE）和百分比误差（PE）；浮动标准差（MovSD）、浮动非正常值患者数（MovSO）用于监测随机误差（RE）
4. 控制限：1. 分布法：通过分析历史数据生成的PBRTQC计算值，得出计算值的均值与标准差，根据正态分布或其它分布特征计算出对应的控制限
           2. 百分位点法：根据目标假阳性报警率（DFAR），如0.1%，考虑双边的情况下，那么控制限则为历史数据PBRTQC计算值的0.05%和99.95%百分位数点

## 1.3、模型评估指标
1. 假阳性报警率（FAR）：指误报次数与样本数的比值，文献中通常要求该值≤0.1%，即每进行1000例患者样本的实时质控，最多允许1次误报，表征模型的特异性，FAR越小，模型特异性越好
2. 误差检出前平均患者样本数（ANPed）：误差出现时至模型检测出误差所经历的患者样本例数，表征模型的灵敏度，例数越少，模型灵敏度越高
3. 二者关系：二者之间存在矛盾关系，FAR越低，ANPed越大，实际应用中应综合考虑，能满足当前项目的需求即可

# 2、实现简要流程

1. 数据类型为信号量，函数模型修改为mq，mq分位数设置为50,转换方式修改为不转换
2. 根据条件查询数据集合(仪器+批号+项目+时间)
3. 定量定性项目过滤，定量项目过滤掉单位为S/CO，定性项目液体类型为C（校准）或单位为S/CO
4. cut-off过滤（不包括校准点数据）
5. 根据washStatus判断是否需要排除修改前后信号值不一致的数据
6. 得到两份数据。allDataList：经过上述步骤剩余的数据。list：经过上述步骤再过滤掉校准点（液体类型C）数据
7. list为空、list大于20000，list大小小于训练数据集样本容量n。直接返回
8. 寻找定标线。液体类型为c（校准）,连续12个点确定一个定标线（不连续的舍弃）
9. 暂存list
10. 遍历函数模型集合，一个模型出一个图
11. 暂存全量数据list
12. 计算控制线
13. 每次移动训练数据集个数个数据。计算截断限LTL和UTL（通过移动分位数mq计算）
14. 根据LTL和UTL截断sublist数据。数据被截断并且是定标点，将前一个点设置为定标点
15. 数据转换sublist，使数据更符合正态性(0-Box-Cox变换、1-对数变换、2-倒数变换、3-平方根变换)
16. 记录最优lambda，加权失控报警0.5N和2N最优lambda用N计算出的lambda
17. 计算本次控制线（LCL,UCL）
18. 判断控制线的合理性：根据控制线偏差和Moving-Slope斜率最大值，最小值判断（斜率判断最新版本已废除）
19. 满足合理性判断时，暂存N转换后的数据,用作失控报警中的加权失控计算（0.5N，2N）
20. 未找到合适控制线时，直接返回并提示用户
21. 使用计算控制线得到的LTL和UTL截断全量数据，得到list
22. list数据转换(0-Box-Cox变换、1-对数变换、2-倒数变换、3-平方根变换)
23. 暂存转换后的数据,用作失控报警中的加权失控计算
24. list统计值计算(0-MA、1-MQ、2-EWMA、3-MovSD、4-MR)
25. Moving-Slope计算系统误差点（最新版本废弃）
26. 失控报警：失控点计算(0-单点失控,1-连续X点失控,2-加权失控，3-加权连续失控)
27. 找到所有定标线
28. 响应封装

# 3、代码实现

## 3.1、基本流程框架

```java
public AjaxResult pbrtqcLineData(PbrtqcDTO dto) {  
    PBRTQCResultVO response = new PBRTQCResultVO();  
    // 请求转换  
    if (dto.getDataType() == 1) {  
        // 数据类型位信号量，函数模型修改为mq，mq分位数设置为50,转换方式修改为不转换  
        dto.setFuncModel(1);  
        dto.setMqQuantile(50.0);  
        dto.setTransType(null);  
    }  
    // 根据条件查询数据集合(仪器+批号+项目+时间)  
    List<PbrtqcWashed> allDataList = iPbrtqcWashedService.selectLineData(dto);  
    // 定量项目&定性项目过滤  
    if (dto.getProjectType() == 0){  
        allDataList = allDataList.stream().filter(item -> !"S/CO".equals(item.getUnit())).collect(Collectors.toList());  
    }else {  
        allDataList = allDataList.stream().filter(item -> "C".equals(item.getLiquidType()) || "S/CO".equals(item.getUnit())).collect(Collectors.toList());  
    }  
    // cut-off过滤(不包括校准点数据)  
    if (1 == dto.getCutOffCondition()){  
        allDataList = allDataList.stream().filter(item -> "C".equals(item.getLiquidType()) ||  item.getValue() < dto.getCutOffValue()).collect(Collectors.toList());  
    }else if (2 == dto.getCutOffCondition()){  
        allDataList = allDataList.stream().filter(item -> "C".equals(item.getLiquidType()) ||  BigDecimal.valueOf(item.getValue()).compareTo(BigDecimal.valueOf(dto.getCutOffValue())) >= 0).collect(Collectors.toList());  
    }  
    // 不包含液体类型为C的数据 根据washStatus判断是否需要排除修改前后信号值不一致的数据，排除时不对C（校准点）排除  
    if (dto.getWashStatus() == 0){  
        allDataList = allDataList.stream().filter(item -> "C".equals(item.getLiquidType()) || item.getModifiedSignal().equals(item.getOriginalSignal())).collect(Collectors.toList());  
    }  
    List<PbrtqcWashed> list = allDataList.stream().filter(item -> !"C".equals(item.getLiquidType())).collect(Collectors.toList());  
    // 数据为空  
    if (CollectionUtils.isEmpty(list)) {  
        return AjaxResult.error("查询数据为空");  
    }  
    // 数据量过大不进行计算  
    if (list.size() > 20000) {  
        return AjaxResult.error("数据量过大（超过20000条），请调整查询条件");  
    }  
    // 训练数据集样本大于集合数量，调整数量  
    if (dto.getN() > list.size()) {  
        return AjaxResult.error("总数据量为：" + list.size() + ",小于训练数据集样本N，请调整N大小!");  
    }  
    // 寻找定标线  
    List<CalibrationPoint> calibrationDetailList = findCalibrationLine(allDataList); // 定标点集合  
    // 暂存全量数据list  
    List<PbrtqcWashed> tempList = saveDataList(list);  
    // 单位记录  
    String unit = list.get(0).getUnit();  
    List<PBRTQCData> pbrtqcDataList = new ArrayList<>();  
    List<Integer> funcModels = Arrays.stream(dto.getFuncModels().split(",")).map(Integer::parseInt).collect(Collectors.toList());  
    // 把movsd放到集合最后一个元素  
    for (int i=0; i< funcModels.size(); i++){  
        if (funcModels.get(i) == 3){  
            funcModels.remove(i);  
            funcModels.add(3);  
        }  
    }  
    // 序号  
    dto.setIndex(0);  
    for (Integer funcModel : funcModels) {  
        List<PbrtqcWashed> modelDataList = saveDataList(list);  
        // 函数模型  
        dto.setFuncModel(funcModel);  
        // 控制限偏差设置  
        if (funcModel == 3) {  
            dto.setLimitQuantile(dto.getSdLimitQuantile());  
        } else {  
            dto.setLimitQuantile(dto.getMqLimitQuantile());  
        }  
        // 计算N控制限  
        ControlLineData controlLine = calculateControlLine(dto, tempList, 1, 0, dto.getIndex());  
        // 控制限判断  
        if ((dto.getIndex() ==  0 && controlLine.getUcl() == null) || (dto.getIndex() > 0 && dto.getFindControlLine() == null)){  
            return AjaxResult.error("未找到合适的控制线,请设置合理的控制限偏差");  
        }  
        List<PbrtqcWashed> cutDataList = modelDataList;  
        if (dto.getCutType() != 0) {  
            // LTL（下截断限值）和UTL（上截断限值）  
            QuantileData quantileData = new QuantileData(dto.getLtl(),dto.getUtl());  
            // 根据截断方式截断数据  
            cutDataList = cutData(dto, quantileData, modelDataList);  
        }  
        // 数据转换(0-Box-Cox变换、1-对数变换、2-倒数变换、3-平方根变换)  
        dataTrans(dto, cutDataList, 0);  
        // 暂存转换后的数据,用作失控报警中的加权失控计算  
        List<PbrtqcWashed> transData = new ArrayList<>();  
        if (dto.getAlarmMode() == 2 || dto.getAlarmMode() == 3){  
            transData = saveDataList(cutDataList);  
        }  
        // 统计值计算(0-MA、1-MQ、2-EWMA、3-MovSD、4-MR)  
        List<PBRTQCLineVO> dataList = funcModelCalculate(dto, cutDataList,1);  
        // Moving-Slope计算系统误差失控点  
        List<PBRTQCLineVO> systemErrorDataList = systemErrorPoint(dto, dataList);  
        // 失控报警  
        List<PBRTQCLineVO> alarmDataList = outOfControlAlarm(dto, tempList, dataList, controlLine, transData);  
        // 批号计算  
        List<String> lots = dataList.stream().map(PBRTQCLineVO::getName).distinct().collect(Collectors.toList());  
        lots.add(0, "失控点");  
        lots.add(1, "系统误差点");  
        // 设置失控点  
        alarmDataList.forEach(item -> dataList.get(item.getId()).setName("失控点"));  
        // 找到所有的定标线  
        List<PBRTQCLineVO> calibrationPointList = dataList.stream().filter(item ->  
                item.getIsCalibration() != null && item.getIsCalibration().equals(1)).collect(Collectors.toList());  
        List<Integer> calibrationList = calibrationPointList.stream().map(PBRTQCLineVO::getId).collect(Collectors.toList());  
        // 寻找失控点中的所有系统误差点  
        List<PBRTQCLineVO> systemErrorData = alarmDataList.stream().filter(item -> systemErrorDataList.stream().anyMatch(obj -> obj.getId().equals(item.getId()))).collect(Collectors.toList());  
        systemErrorData.forEach(item -> dataList.get(item.getId()).setName("系统误差点"));  
        // 响应封装  
        PBRTQCData data = new PBRTQCData();  
        data.setList(dataList);  
        data.setLots(lots);  
        data.setUnit(unit);  
        data.setMethodName(DictUtils.getDictLabel("ims_pbrtqc_func_model", dto.getFuncModel().toString()));  
        data.setAlarmList(alarmDataList);  
        data.setSystemErrorList(systemErrorData);  
        data.setLcl(controlLine.getLcl());  
        data.setUcl(controlLine.getUcl());  
        data.setOriginLcl(controlLine.getOriginLcl());  
        data.setOriginUcl(controlLine.getOriginUcl());  
        data.setCalibrationList(calibrationList);  
        pbrtqcDataList.add(data);  
        dto.setIndex(dto.getIndex() + 1);  
    }  
    response.setList(pbrtqcDataList);  
    response.setLtl(dto.getLtl());  
    response.setUtl(dto.getUtl());  
    response.setCalibrationPointList(calibrationDetailList);  
    return AjaxResult.success(response);  
}
```


## 3.2、控制线计算流程

```java
private ControlLineData calculateControlLine(PbrtqcDTO dto, List<PbrtqcWashed> list, Integer transFlag, Integer alarmFlag, Integer index) {  
//        List<PbrtqcWashed> list = saveDataList(originlist);  
        int size = list.size();  
        Integer offset = dto.getLimitOffset();  
        Integer n = dto.getN();  
        Double limitQuantile = dto.getLimitQuantile();  
        ControlLineData controlLineData = new ControlLineData();  
        // 第二次计算控制线用第一次的截断转换数据  
        if(index > 0){  
            // 计算控制线  
            ControlLineData controlLine =  calculateCurrentControlLine(dto.getTransDataList(), dto);  
            Double ucl = controlLine.getUcl();  
            Double lcl = controlLine.getLcl();  
            Double originLcl = controlLine.getOriginLcl();  
            Double originUcl = controlLine.getOriginUcl();  
            // 新增控制线判断条件Moving-Slope  
            boolean msFlag = true;  
            if (dto.getMinSlope() != null && dto.getMaxSlope() != null){  
                msFlag = movingSlope(dto, dto.getStatisticList2());  
            }  
            // 判断合理性  
            if ((originUcl - originLcl) / (originLcl + originUcl) <= limitQuantile / 100 && msFlag) {  
                dto.setFindControlLine(1);  
                controlLineData.setLcl(lcl);  
                controlLineData.setUcl(ucl);  
                controlLineData.setOriginUcl(originUcl);  
                controlLineData.setOriginLcl(originLcl);  
                dto.setBestMean2(dto.getUseMean2());  
                dto.setBestSD2(dto.getUseSD2());  
                return controlLineData;  
            }  
        }  
        for (int i = 0; i < size - offset; i++) {  
            // 偏移次数超过100次，说明偏差限制选择的不合理，直接跳出  
            if (i > 100) {  
                break;  
            }  
            List<PbrtqcWashed> subList = list.stream().skip(offset * i).limit(n).collect(Collectors.toList());  
            subList = saveDataList(subList);  
            if (subList.size() < n) {  
                return controlLineData;  
            }  
            QuantileData quantileData = new QuantileData();  
            if (dto.getCutType() != 0) {  
                // 计算LTL（下截断限值）和UTL（上截断限值）  
                quantileData = calculateQuantile(subList, dto);  
                // 训练数据集截断后数据  
                subList = cutData(dto, quantileData, subList);  
            }  
            // 数据转换(0-Box-Cox变换、1-对数变换、2-倒数变换、3-平方根变换)  
            List<PbrtqcWashed> transDatas = dataTrans(dto, subList, transFlag);  
            // 暂存N转换后的数据，用来计算加权失控0.5N，2N  
            List<PbrtqcWashed> tempTransDataList = new ArrayList<>();  
            if (dto.getAlarmMode() == 2 || dto.getAlarmMode() == 3) {  
                tempTransDataList = saveDataList(transDatas);  
            }  
            // 计算本次控制线  
            ControlLineData currentControlLine =  calculateCurrentControlLine(transDatas, dto);  
            // 计算合理性  
            Double ucl = currentControlLine.getUcl();  
            Double lcl = currentControlLine.getLcl();  
            Double originLcl = currentControlLine.getOriginLcl();  
            Double originUcl = currentControlLine.getOriginUcl();  
            // 新增控制线判断条件Moving-Slope  
            boolean msFlag = true;  
            if (dto.getMinSlope() != null && dto.getMaxSlope() != null){  
                msFlag = movingSlope(dto, dto.getStatisticList());  
            }  
            if ((originUcl - originLcl) / (originLcl + originUcl) <= limitQuantile / 100 && msFlag) {  
                if (index == 0){  
                    // 记录截断限，用于第二次全量数据的截断  
                    dto.setLtl(quantileData.getLtl());  
                    dto.setUtl(quantileData.getUtl());  
                    // 记录该次转换后的数据,用于第二次统计量的计算  
                    dto.setTransDataList(transDatas);  
                    dto.setBestMean(dto.getUseMean());  
                    dto.setBestSD(dto.getUseSD());  
                }  
                controlLineData.setLcl(lcl);  
                controlLineData.setUcl(ucl);  
                controlLineData.setOriginUcl(originUcl);  
                controlLineData.setOriginLcl(originLcl);  
                // N转换后的数据，用于加权失控计算  
                dto.setTransDataN(tempTransDataList);  
                break;            }  
        }  
        return controlLineData;  
    }
```


## 3.3、数据转换box-cox转换

```java
private ControlLineData calculateControlLine(PbrtqcDTO dto, List<PbrtqcWashed> list, Integer transFlag, Integer alarmFlag, Integer index) {  
//        List<PbrtqcWashed> list = saveDataList(originlist);  
        int size = list.size();  
        Integer offset = dto.getLimitOffset();  
        Integer n = dto.getN();  
        Double limitQuantile = dto.getLimitQuantile();  
        ControlLineData controlLineData = new ControlLineData();  
        // 第二次计算控制线用第一次的截断转换数据  
        if(index > 0){  
            // 计算控制线  
            ControlLineData controlLine =  calculateCurrentControlLine(dto.getTransDataList(), dto);  
            Double ucl = controlLine.getUcl();  
            Double lcl = controlLine.getLcl();  
            Double originLcl = controlLine.getOriginLcl();  
            Double originUcl = controlLine.getOriginUcl();  
            // 新增控制线判断条件Moving-Slope  
            boolean msFlag = true;  
            if (dto.getMinSlope() != null && dto.getMaxSlope() != null){  
                msFlag = movingSlope(dto, dto.getStatisticList2());  
            }  
            // 判断合理性  
            if ((originUcl - originLcl) / (originLcl + originUcl) <= limitQuantile / 100 && msFlag) {  
                dto.setFindControlLine(1);  
                controlLineData.setLcl(lcl);  
                controlLineData.setUcl(ucl);  
                controlLineData.setOriginUcl(originUcl);  
                controlLineData.setOriginLcl(originLcl);  
                dto.setBestMean2(dto.getUseMean2());  
                dto.setBestSD2(dto.getUseSD2());  
                return controlLineData;  
            }  
        }  
        for (int i = 0; i < size - offset; i++) {  
            // 偏移次数超过100次，说明偏差限制选择的不合理，直接跳出  
            if (i > 100) {  
                break;  
            }  
            List<PbrtqcWashed> subList = list.stream().skip(offset * i).limit(n).collect(Collectors.toList());  
            subList = saveDataList(subList);  
            if (subList.size() < n) {  
                return controlLineData;  
            }  
            QuantileData quantileData = new QuantileData();  
            if (dto.getCutType() != 0) {  
                // 计算LTL（下截断限值）和UTL（上截断限值）  
                quantileData = calculateQuantile(subList, dto);  
                // 训练数据集截断后数据  
                subList = cutData(dto, quantileData, subList);  
            }  
            // 数据转换(0-Box-Cox变换、1-对数变换、2-倒数变换、3-平方根变换)  
            List<PbrtqcWashed> transDatas = dataTrans(dto, subList, transFlag);  
            // 暂存N转换后的数据，用来计算加权失控0.5N，2N  
            List<PbrtqcWashed> tempTransDataList = new ArrayList<>();  
            if (dto.getAlarmMode() == 2 || dto.getAlarmMode() == 3) {  
                tempTransDataList = saveDataList(transDatas);  
            }  
            // 计算本次控制线  
            ControlLineData currentControlLine =  calculateCurrentControlLine(transDatas, dto);  
            // 计算合理性  
            Double ucl = currentControlLine.getUcl();  
            Double lcl = currentControlLine.getLcl();  
            Double originLcl = currentControlLine.getOriginLcl();  
            Double originUcl = currentControlLine.getOriginUcl();  
            // 新增控制线判断条件Moving-Slope  
            boolean msFlag = true;  
            if (dto.getMinSlope() != null && dto.getMaxSlope() != null){  
                msFlag = movingSlope(dto, dto.getStatisticList());  
            }  
            if ((originUcl - originLcl) / (originLcl + originUcl) <= limitQuantile / 100 && msFlag) {  
                if (index == 0){  
                    // 记录截断限，用于第二次全量数据的截断  
                    dto.setLtl(quantileData.getLtl());  
                    dto.setUtl(quantileData.getUtl());  
                    // 记录该次转换后的数据,用于第二次统计量的计算  
                    dto.setTransDataList(transDatas);  
                    dto.setBestMean(dto.getUseMean());  
                    dto.setBestSD(dto.getUseSD());  
                }  
                controlLineData.setLcl(lcl);  
                controlLineData.setUcl(ucl);  
                controlLineData.setOriginUcl(originUcl);  
                controlLineData.setOriginLcl(originLcl);  
                // N转换后的数据，用于加权失控计算  
                dto.setTransDataN(tempTransDataList);  
                break;            }  
        }  
        return controlLineData;  
    }

/**  
 * 计算最优的lambda值  
 *  
 * @param data 数据  
 * @return 最优的lambda值  
 */  
public static double findBestLambda(double[] data) {  
    // 定义lambda的范围和步长  
    double lambdaStart = -5.0;  
    double lambdaEnd = 5.0;  
    double lambdaStep = 0.01;  
  
    // 计算原始数据的几何均值G  
    double G = 1.0;  
    for (double x : data) {  
        G *= Math.pow(x, 1.0 / data.length);  
    }  
  
    // 计算最优lambda  
    double minSD = Double.MAX_VALUE;  
    double minLambda = -5;  
    for (double lambda = lambdaStart; lambda <= lambdaEnd; lambda += lambdaStep) {  
        double[] transformedData = boxCoxTransform(data, lambda, G);  
        double avg = calculateAverage(transformedData);  
        double sd = calculateStandardDeviation(transformedData, avg);  
        if (sd < minSD) {  
            minSD = sd;  
            minLambda = lambda;  
        }  
    }  
    return minLambda;  
}

/**  
 * 计算Box-Cox变换后的数据  
 * @param data   数据  
 * @param lambda lambda值  
 * @return Box-Cox变换后的数据  
 */  
public static double[] boxCoxTransform(double[] data, double lambda, double G) {  
    double[] transformedData = new double[data.length];  
    if (lambda == 0) {  
        for (int i = 0; i < data.length; i++) {  
            transformedData[i] = G * Math.log(data[i]);  
        }  
    } else {  
        for (int i = 0; i < data.length; i++) {  
            transformedData[i] = (Math.pow(data[i], lambda) - 1) / (lambda * Math.pow(G, lambda - 1));  
        }  
    }  
    return transformedData;  
}
```

## 3.4、统计值计算之mq

```java
/**  
 * 移动分位数（MQ）  
 * @param N 窗口大小  
 * @param X mq分位数  
 * @param list  
 * @return  
 */  
public static List<PBRTQCLineVO> MQ(Integer N, Double X, List<PbrtqcWashed> list) {  
    List<PBRTQCLineVO> result = new ArrayList<>();  
    for (int i = 0; i < list.size() -N + 1; i++) {  
        double val1 = (N + 1) * X / 100;  
        // 整数部分  
        int j = (int) val1;  
        // 小数部分  
        double g = val1 - j;  
        List<Integer> subIndex = new ArrayList<>();  
        double calValue;  
        // list从小到大排序  
        List<PbrtqcWashed> subList = list.stream().skip(i).limit(N).collect(Collectors.toList());  
        List<PbrtqcWashed> collect = subList.stream().sorted(Comparator.comparing(PbrtqcWashed::getValue)).collect(Collectors.toList());  
        if (g == 0) {  
            calValue = collect.get(j - 1).getValue();  
        } else {  
            calValue = (1 - g) * collect.get(j - 1).getValue() + g * collect.get(j).getValue();  
        }  
        PBRTQCLineVO vo = new PBRTQCLineVO();  
        vo.setId(i);  
        vo.setName(list.get(i+N-1).getReagentBatch());  
        vo.setValue(calValue);  
        vo.setIsCalibration(list.get(i+N-1).getIsCalibration());  
        vo.setReadyTime(list.get(i+N-1).getReadyTime());  
        result.add(vo);  
    }  
    return result;  
}
```


# 4、效果图

![PBRTQC](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202310081549987.png)
