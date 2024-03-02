---
title: sharding-jdbc实现自动按月分表
date: 2023/09/16
categories:
  - coding
tags:
  - sharding-jdbc
  - 分库分表
  - 技术文章
abbrlink: 53776
---

# 1、基本需求

1、 项目中我们希望 能够按照时间、类别来添加表。但是sharding-jdbc 是固定配置 的 actual-data-nodes 参数。也就是说我们需要提前创建好分表或者分库。那么我们需要如何来实现动态创建表，并且动态刷新 actual-data-nodes 呢。

2、思路就是写个定时器来动态创建表 ，在创建表的时候 动态刷新 actual-data-nodes 实现动态创建表被shard加载。

# 2、功能实现

## 2.1、添加依赖
```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.0.0-RC1</version>
</dependency>
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>4.0.0-RC1</version>
</dependency>
```


## 2.2、yml配置

```yml
sharding:  
    # 配置绑定表  
    binding-tables[0]: ims_test_result,ims_test_sample_fetch,ims_test_sample_diluent,ims_test_reagent_add,ims_test_ls_add,ims_test_incubate,ims_test_read  
    tables:  
		ims_sample_base:  
		    actual-data-nodes: m1.sharding_data_nodes_2022  
		    key-generator:  
		        column: id  
		        type: SNOWFLAKE  
		        props:  
		            worker.id: ${workerId}  
		    table-strategy:  
		        complex:  
		            sharding-columns: submit_work_time,sample_uid  
			            algorithm-class-name: com.chivd.common.algorithm.TableShardingSampleAlgorithm
```

## 2.3、具体分片算法

```java
public class TableShardingSampleAlgorithm implements ComplexKeysShardingAlgorithm<String> {  
  
    private static final String COLUMN_SAMPLE_UID = "sample_uid";  
  
    private static final String COLUMN_SUBMIT_TIME = "submit_work_time";  
  
    @Override  
    public Collection<String> doSharding(Collection<String> collection, ComplexKeysShardingValue<String> complexKeysShardingValue) {  
        // 逻辑表名  
        String logicTableName = complexKeysShardingValue.getLogicTableName();  
  
        // 精准工单提交时间  
        Collection<String> submitWorkTimeCollection = complexKeysShardingValue.getColumnNameAndShardingValuesMap().getOrDefault(COLUMN_SUBMIT_TIME, new ArrayList<>());  
        ArrayList<String> submitWorkTimeList = new ArrayList<>(submitWorkTimeCollection);  
        if (CollectionUtils.isNotEmpty(submitWorkTimeList)) {  
            Set<String> set = new HashSet<>();  
            for (int i = 0; i < submitWorkTimeList.size(); i++) {  
                Date date = DateUtils.parseDate(submitWorkTimeList.get(i));  
                StringBuffer tableName = new StringBuffer();  
                tableName.append(logicTableName).append("_").append(DateUtils.parseDateToStr(DateUtils.YYYYMM, date));  
                set.add(tableName.toString());  
            }  
            return set;  
        }  
  
        // 工单提交时间范围  
        Range<String> submitWorkTimeRange = complexKeysShardingValue.getColumnNameAndRangeValuesMap().get(COLUMN_SUBMIT_TIME);  
        if (submitWorkTimeRange != null) {  
            // 实体表名集合  
            Set<String> result = new HashSet<>();  
            Date lowerDate = DateUtils.parseDate(submitWorkTimeRange.lowerEndpoint());  
            Date upperDate = DateUtils.parseDate(submitWorkTimeRange.upperEndpoint());  
            // 获取月份间隔  
            int monthSpace = DateUtils.getMonthSpace(lowerDate, upperDate);  
            // 获取所有的实体表名  
            for (int i = 0; i < monthSpace; i++) {  
                result.add(logicTableName + "_" + DateUtils.parseDateToStr(DateUtils.YYYYMM, lowerDate));  
                lowerDate = DateUtils.addMonths(lowerDate, 1);  
            }  
            return result;  
        }  
  
        // 样本uid  
        Collection<String> testUids = complexKeysShardingValue.getColumnNameAndShardingValuesMap().getOrDefault(COLUMN_SAMPLE_UID, new ArrayList<>());  
        if (CollectionUtils.isNotEmpty(testUids)) {  
            return testUids.stream().map(uid -> {  
                String[] split = uid.split("-");  
                StringBuffer tableName = new StringBuffer();  
                tableName.append(logicTableName)  
                        .append("_")  
                        .append(split[split.length - 1].substring(0,6));  
                return tableName.toString();  
            }).collect(Collectors.toSet());  
        }  
        return null;  
    }  
}
```
## 2.4、创建配置表

配置表包含所有需要分表的逻辑表表名，分表开始年月等信息

```sql
CREATE TABLE `ims_sharding_config` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `table_name` varchar(100) DEFAULT NULL COMMENT '表名',
  `start_year_month` varchar(20) DEFAULT NULL COMMENT '分表开始年月',
  `comment` varchar(100) DEFAULT NULL COMMENT '备注',
  `is_deleted` tinyint(4) DEFAULT '0' COMMENT '(0-未删除 1-删除)',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=21 DEFAULT CHARSET=utf8mb4;
```

## 2.5、创建定时任务

1. 启动时自动刷新actual-data-nodes节点
2. 自动创建下月分表

```java
@Slf4j  
@Component  
public class ShardingTableUtils {  
    @Autowired  
    DataSource dataSource;  
  
    @Autowired  
    IShardingConfigService shardingConfigService;  
  
    /**  
     * @Description 项目启动时刷新节点配置  
     **/    
     @PostConstruct  
    public void startRefresh() {  
        AutoCreateTable();  
    }  
  
    /**  
     * @Description 刷新actual-data-nodes节点配置  
     **/    
     public void actualTablesRefresh() {  
        log.info("-------------- 开始刷新sharding配置 ---------------");  
        try {  
            List<ShardingConfig> shardingConfigList = shardingConfigService.listShardingConfig();  
            ShardingDataSource dataSource = (ShardingDataSource) this.dataSource;  
            if (shardingConfigList == null || shardingConfigList.isEmpty()) {  
                log.info("【sharding自动配置】配置表为空");  
                return;            }  
            String curYearAndMonth = DateUtils.getYearAndMonth(DateUtils.monthAdd(new Date(),1).getTime());  
            Field modifiersField = Field.class.getDeclaredField("modifiers");  
            modifiersField.setAccessible(true);  
            for (ShardingConfig item : shardingConfigList) {  
                TableRule tableRule = null;  
                tableRule = dataSource.getRuntimeContext().getRule().getTableRule(item.getTableName());  
                List<DataNode> dataNodes = tableRule.getActualDataNodes();  
                String dataSourceName = dataNodes.get(0).getDataSourceName();  
                List<String> monthBetween = getMonthBetween(item.getStartYearMonth(), curYearAndMonth);  
                List<DataNode> newDataNodes = monthBetween.stream()  
                        .map(yearMonth -> new DataNode(dataSourceName + "." + item.getTableName()  
                                + "_" + yearMonth)).collect(Collectors.toList());  
                // 修改actualDataNodesField  
                Field actualDataNodesField = TableRule.class.getDeclaredField("actualDataNodes");  
                actualDataNodesField.setAccessible(true);  
                modifiersField.setInt(actualDataNodesField, actualDataNodesField.getModifiers() & ~Modifier.FINAL);  
                actualDataNodesField.set(tableRule, newDataNodes);  
                // 修改actualTablesField  
                Set<String> actualTables = Sets.newHashSet();  
                Map<DataNode, Integer> dataNodeIndexMap = Maps.newHashMap();  
                AtomicInteger index = new AtomicInteger(0);  
                newDataNodes.forEach(dataNode -> {  
                    actualTables.add(dataNode.getTableName());  
                    if (index.intValue() == 0) {  
                        dataNodeIndexMap.put(dataNode, 0);  
                    } else {  
                        dataNodeIndexMap.put(dataNode, index.intValue());  
                    }  
                    index.incrementAndGet();  
                });  
                Field actualTablesField = TableRule.class.getDeclaredField("actualTables");  
                actualTablesField.setAccessible(true);  
                actualTablesField.set(tableRule, actualTables);  
                // 动态刷新 dataNodeIndexMapField                Field dataNodeIndexMapField = TableRule.class.getDeclaredField("dataNodeIndexMap");  
                dataNodeIndexMapField.setAccessible(true);  
                dataNodeIndexMapField.set(tableRule, dataNodeIndexMap);  
                // 动态刷新 datasourceToTablesMapField                Map<String, Collection<String>> datasourceToTablesMap = Maps.newHashMap();  
                datasourceToTablesMap.put(dataSourceName, actualTables);  
                Field datasourceToTablesMapField = TableRule.class.getDeclaredField("datasourceToTablesMap");  
                datasourceToTablesMapField.setAccessible(true);  
                datasourceToTablesMapField.set(tableRule, datasourceToTablesMap);  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
            log.info("【sharding自动配置】异常" + e.getMessage());  
        }  
    }  
  
    /**  
     * @Description 自动创建不存在的表  
     **/    
     private void AutoCreateTable() {  
        log.info("-------------- 开始创建分表 ---------------");  
        List<ShardingConfig> shardingConfigList = shardingConfigService.listShardingConfig();  
        if (shardingConfigList == null || shardingConfigList.isEmpty()) {  
            log.info("【sharding自动配置】配置表为空");  
            return;        }  
        String curYearAndMonth = DateUtils.getYearAndMonth(DateUtils.monthAdd(new Date(),2).getTime());  
        for (ShardingConfig item : shardingConfigList) {  
            List<String> monthBetween = new ArrayList<>();  
            try {  
                monthBetween = getMonthBetween(item.getStartYearMonth(), curYearAndMonth);  
            } catch (ParseException e) {  
                log.info("【sharding自动配置】日期转化失败" + e.getMessage());  
                e.printStackTrace();  
            }  
            // todo (CREATE TABLE if not exists xx like xxx 表存在会抛异常。为啥呢 这里先catch住让代码继续运行)  
            monthBetween.forEach(yearMonth -> {  
                try {  
                    shardingConfigService.createTable(item.getTableName(), item.getTableName() + "_" + yearMonth);  
                }catch (Exception ignored){}  
            });  
        }  
        // 刷新配置  
        actualTablesRefresh();  
    }  
  
    /**  
     * 获取两个月份之间的所有月份  
     * @param minDate  
     * @param maxDate  
     * @return  
     * @throws ParseException  
     */    private static List<String> getMonthBetween(String minDate, String maxDate) throws ParseException {  
        ArrayList<String> result = new ArrayList<>();  
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMM");//格式化为年月  
  
        Calendar min = Calendar.getInstance();  
        Calendar max = Calendar.getInstance();  
  
        min.setTime(sdf.parse(minDate));  
        min.set(min.get(Calendar.YEAR), min.get(Calendar.MONTH), 1);  
  
        max.setTime(sdf.parse(maxDate));  
        max.set(max.get(Calendar.YEAR), max.get(Calendar.MONTH), 2);  
  
        Calendar curr = min;  
        while (curr.before(max)) {  
            result.add(sdf.format(curr.getTime()));  
            curr.add(Calendar.MONTH, 1);  
        }  
        return result;  
    }  
  
    /**  
     * 每月27号1点自动刷新配置节点  
     */  
    @Scheduled(cron = "0 0 1 27 * ?")  
    public void refreshScheduled() {  
        AutoCreateTable();  
    }  
}
```