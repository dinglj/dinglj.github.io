---
layout : post
title : "大表拆分"
category : 大数据
duoshuo: true
date : 2020-03-08
tags : [mysql,大表拆分,大数据]
---
### 指导原则
1. 通过归档task将业务表数据同步至冷表；冷表保留全量数据，热表保留（1-3）个月数据
2. 通过数据同步工具实时同步至报表库（单独部署）,报表库通过COLD表查询；

1. 字典表： (键分表)
2. 流水表： (时间归档)
3. 组合分表：  (键值水平分表+时间归档)


### 财务凭证大表拆分
1. 按公司维度作为分表key
2. 保留最近2年的数据
3. 2年之前的数据通过job存档到另外一个仓库

### 具体操作
1. 建表 建表规则 vocher_header_{comp_code} vocher_line_{comp_code}
2. 新增后台维护建立新公司表 （对应新增的公司单据先进入异常单） 新增公司表是否会对归档有影响？
3. DML操作改造  对数据库的所有操作都要带comp_code
    - 3.1 凭证冲销
    - 3.2 凭证废弃
    - 3.3 凭证查询
    - 3.4 凭证导出
    - 3.5 凭证创建
    - 3.6 sap回调  并没有comp_code关联？
4. 查询改造
5. 数据归档改造 归档表规则 vocher_header_{comp_code}_{YYYY} vocher_line_{comp_code}_{YYYY}
5. 业务历史数据迁移
6. 归档历史数据查询接口

mysql复制表结构 命令
CREATE TABLE csx_b2b_finance.voucher_header_2211 LIKE csx_b2b_finance.voucher_header;



精确分片算法
对应PreciseShardingAlgorithm，用于处理使用单一键作为分片键的=与IN进行分片的场景。需要配合StandardShardingStrategy使用。


1. 分库分表配置
```
spring.shardingsphere.sharding.tables.voucher_header.actual-data-nodes=ds0.voucher_header_$->{['1933','2115','2116','2121','2126','2127','2128','2129','2130','2207','2210','2211','2304','2408','2814','2815','3505','3750','3751']}
spring.shardingsphere.sharding.tables.voucher_header.table-strategy.standard.sharding-column=comp_code
spring.shardingsphere.sharding.tables.voucher_header.table-strategy.standard.precise-algorithm-class-name=com.yh.csx.finance.dao.sharding.TableShardingAlgorithm
spring.shardingsphere.sharding.tables.voucher_line.actual-data-nodes=ds0.voucher_line_$->{['1933','2115','2116','2121','2126','2127','2128','2129','2130','2207','2210','2211','2304','2408','2814','2815','3505','3750','3751']}
spring.shardingsphere.sharding.tables.voucher_line.table-strategy.standard.sharding-column=comp_code
spring.shardingsphere.sharding.tables.voucher_line.table-strategy.standard.precise-algorithm-class-name=com.yh.csx.finance.dao.sharding.TableShardingAlgorithm
spring.shardingsphere.sharding.binding-tables=voucher_header,voucher_line

spring.shardingsphere.sharding.master-slave-rules.ds0.master-data-source-name=master-0
spring.shardingsphere.sharding.master-slave-rules.ds0.slave-data-source-names=slave-0

spring.shardingsphere.sharding.master-slave-rules.ds0.load-balance-algorithm-type=round_robin
```


## 具体实操
1. 数据 ->  vocher_header_{comp_code} -> binlog -> 中间件 -> vocher_header_{shardingkey % n} (冗余用于查询)
2. 数据 ->  vocher_header_{comp_code} ->  vocher_header_{shardingkey % n} (冗余用于查询)
3. 数据更新插入删除操作一定是精准分片  查询操作有可能是全表扫描

## 在做具体方案的时候发现凭证号规则含有公司编码，进一步简化











## sharding-jdbc 执行一次sql大概流程
1. 先解析SQL生成sqlStatement
2. 获取SQL分片条件ShardingConditions
3. 根据分片规则生成路由节点（包含实际库和表）
4. 根据路由的实际节点重写SQL
5. 使用反射设置SQL执行参数
6. 执行SQL（交由第三方数据库连接池执行）
7. 结果处理（交由mybatis处理）

ShardingDataSourceFactory#createDataSource -> 
1. datasource 初始化
2. ShardingConnection
    - 2.1 ShardingDataSource#getConnection
3. PreparedStatementRoutingEngine
4. PreparedStatementExecutor
5. BatchPreparedStatementExecutor
6. SQLRouteResult
7. ResultSet

ShardingPreparedStatement#execute -> PreparedStatementRoutingEngine#route -> ParsingSQLRouter#parse -> SQLParsingEngine#parse -> SQLParser#parse(=sqlStatement) -> StandardRoutingEngine#route -> StandardRoutingEngine#routeByShardingConditions -> StandardRoutingEngine#route


## 初始化数据源
1. 解析配置初始化初始化所有数据源
```
sharding.jdbc.datasource.names=ngbmp,ngbmpbj,ngbmptj,ngbmpnm
######没有进行分库分表的查询，使用默认数据源
###########################数据源配置##################################
#数据库连接池类名称
sharding.jdbc.datasource.ngbmp.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ngbmp.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ngbmp.url=jdbc:mysql://localhost:3306/ngbmp?characterEncoding=UTF-8&serverTimezone=UTC
sharding.jdbc.datasource.ngbmp.username=root
sharding.jdbc.datasource.ngbmp.password= Aa123456

sharding.jdbc.datasource.ngbmpbj.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ngbmpbj.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ngbmpbj.url=jdbc:mysql://localhost:3306/ngbmp_bj?characterEncoding=UTF-8&serverTimezone=UTC
sharding.jdbc.datasource.ngbmpbj.username=root
sharding.jdbc.datasource.ngbmpbj.password= Aa123456

sharding.jdbc.datasource.ngbmptj.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ngbmptj.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ngbmptj.url=jdbc:mysql://localhost:3306/ngbmp_tj?characterEncoding=UTF-8&serverTimezone=UTC
sharding.jdbc.datasource.ngbmptj.username=root
sharding.jdbc.datasource.ngbmptj.password= Aa123456

sharding.jdbc.datasource.ngbmpnm.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ngbmpnm.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ngbmpnm.url=jdbc:mysql://localhost:3306/ngbmp_nm?characterEncoding=UTF-8&serverTimezone=UTC
sharding.jdbc.datasource.ngbmpnm.username=root
sharding.jdbc.datasource.ngbmpnm.password= Aa123456

@SuppressWarnings("unchecked")
private void setDataSourceMap(final Environment environment) {
String prefix = "sharding.jdbc.datasource.";
String dataSources = environment.getProperty(prefix + "names");
for (String each : dataSources.split(",")) {
    try {
        Map<String, Object> dataSourceProps = PropertyUtil.handle(environment, prefix + each, Map.class);
        Preconditions.checkState(!dataSourceProps.isEmpty(), "Wrong datasource properties!");
        DataSource dataSource = DataSourceUtil.getDataSource(dataSourceProps.get("type").toString(), dataSourceProps);
        dataSourceMap.put(each, dataSource);
    } catch (final ReflectiveOperationException ex) {
        throw new ShardingException("Can't find datasource type!", ex);
    }
}
}
```
2. 根据不同分片规则配置生成不同数据源包装类 SpringBootConfiguration#dataSource
```
@Bean
public DataSource dataSource() throws SQLException {
    if (null != masterSlaveProperties.getMasterDataSourceName()) {
        return MasterSlaveDataSourceFactory.createDataSource(dataSourceMap, masterSlaveSwapper.swap(masterSlaveProperties), propMapProperties.getProps());
    }
    if (!encryptProperties.getEncryptors().isEmpty()) {
        return EncryptDataSourceFactory.createDataSource(dataSourceMap.values().iterator().next(), encryptSwapper.swap(encryptProperties));
    }
    return ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingSwapper.swap(shardingProperties), propMapProperties.getProps());
}
```


## 读写分离
1. 配置
```
spring.shardingsphere.enabled=true
#数据源名称，多数据源以逗号分隔
spring.shardingsphere.datasource.names=ngbmp,ngbmpbj,ngbmptj,ngbmpnm
######没有进行分库分表的查询，使用默认数据源
###########################数据源配置##################################
#数据库连接池类名称
spring.shardingsphere.datasource.ngbmp.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ngbmp.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ngbmp.url=jdbc:mysql://localhost:3306/ngbmp?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ngbmp.username=root
spring.shardingsphere.datasource.ngbmp.password= Aa123456

spring.shardingsphere.datasource.ngbmpbj.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ngbmpbj.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ngbmpbj.url=jdbc:mysql://localhost:3306/ngbmp_bj?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ngbmpbj.username=root
spring.shardingsphere.datasource.ngbmpbj.password= Aa123456

spring.shardingsphere.datasource.ngbmptj.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ngbmptj.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ngbmptj.url=jdbc:mysql://localhost:3306/ngbmp_tj?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ngbmptj.username=root
spring.shardingsphere.datasource.ngbmptj.password= Aa123456

spring.shardingsphere.datasource.ngbmpnm.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ngbmpnm.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ngbmpnm.url=jdbc:mysql://localhost:3306/ngbmp_nm?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ngbmpnm.username=root
spring.shardingsphere.datasource.ngbmpnm.password= Aa123456

spring.shardingsphere.masterslave.load-balance-algorithm-type = round_robin
spring.shardingsphere.masterslave.name = master-0
spring.shardingsphere.masterslave.master-data-source-name = ngbmp
spring.shardingsphere.masterslave.slave-data-source-names = ngbmpbj,ngbmptj,ngbmpnm
spring.shardingsphere.props.sql.show = true
```
2. 数据源初始化 构建的数据源是sharding封装的数据源和连接 MasterSlaveConnection
```
// SpringBootConfiguration#dataSource 
@Bean
public DataSource dataSource() throws SQLException {
    //根据读写分离配置初始化 MasterSlaveDataSource
    if (null != masterSlaveProperties.getMasterDataSourceName()) {
        return MasterSlaveDataSourceFactory.createDataSource(dataSourceMap, masterSlaveSwapper.swap(masterSlaveProperties), propMapProperties.getProps());
    }
    if (!encryptProperties.getEncryptors().isEmpty()) {
        return EncryptDataSourceFactory.createDataSource(dataSourceMap.values().iterator().next(), encryptSwapper.swap(encryptProperties));
    }
    return ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingSwapper.swap(shardingProperties), propMapProperties.getProps());
}
```
3.  执行查询 SimpleExecutor.doQuery -> RoutingStatementHandler.prepare -> PreparedStatementHandler.instantiateStatement -> MasterSlaveConnection.prepareStatement
    - 3.1 在mybatis中获取statement的时候是通过封装的数据源获取封装连接MasterSlaveConnection
    - 3.2 在MasterSlaveConnection通过负载规则路由到对应库 生成PreparedStatement 如下
```
//通过connection包装类获取MasterSlavePreparedStatement
public MasterSlavePreparedStatement(
        final MasterSlaveConnection connection, final String sql, final int resultSetType, final int resultSetConcurrency, final int resultSetHoldability) throws SQLException {
    this.connection = connection;
    masterSlaveRouter = new MasterSlaveRouter(connection.getMasterSlaveDataSource().getMasterSlaveRule(),
            connection.getMasterSlaveDataSource().getShardingProperties().<Boolean>getValue(ShardingPropertiesConstant.SQL_SHOW));
    for (String each : masterSlaveRouter.route(sql)) {
        PreparedStatement preparedStatement = connection.getConnection(each).prepareStatement(sql, resultSetType, resultSetConcurrency, resultSetHoldability);
        routedStatements.add(preparedStatement);
    }
}
//通过规则路由到对应表  MasterSlavePreparedStatement#route
private Collection<String> route(final SQLType sqlType) {
    if (isMasterRoute(sqlType)) {
        MasterVisitedManager.setMasterVisited();
        return Collections.singletonList(masterSlaveRule.getMasterDataSourceName());
    }
    return Collections.singletonList(masterSlaveRule.getLoadBalanceAlgorithm().getDataSource(
            masterSlaveRule.getName(), masterSlaveRule.getMasterDataSourceName(), new ArrayList<>(masterSlaveRule.getSlaveDataSourceNames())));
}
//路由规则 RoundRobinMasterSlaveLoadBalanceAlgorithm
@Override
public String getDataSource(final String name, final String masterDataSourceName, final List<String> slaveDataSourceNames) {
    AtomicInteger count = COUNTS.containsKey(name) ? COUNTS.get(name) : new AtomicInteger(0);
    COUNTS.putIfAbsent(name, count);
    count.compareAndSet(slaveDataSourceNames.size(), 0);
    return slaveDataSourceNames.get(Math.abs(count.getAndIncrement()) % slaveDataSourceNames.size());
}
```

## 分库分表


## 精准分片 + 读写分离
```
spring.shardingsphere.sharding.tables.test.actual-data-nodes=$->{['ds0']}.test_20190$->{4..6}
spring.shardingsphere.sharding.tables.test.table-strategy.standard.sharding-column=stat_date
spring.shardingsphere.sharding.tables.test.table-strategy.standard.precise-algorithm-class-name=com.nullyb.springbootshardingjdbc.algorithm.TableShardingAlgorithm
spring.shardingsphere.sharding.master-slave-rules.ds0.master-data-source-name=ngbmp
spring.shardingsphere.sharding.master-slave-rules.ds0.slave-data-source-names=ngbmpbj, ngbmptj, ngbmpnm
```
