1. 定义索引
```
package me.david;

import lombok.AllArgsConstructor;
import lombok.Getter;

@AllArgsConstructor
public enum EsIndex {
    INDEX_INTELLIGENT("intelligent_index"),
    @Getter
    private String name;
}
```
2. 定义entity
```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CollectLabel {
    private String id;
    private String labelId;
    private String email;
    private String label;
    private String createAt;
    private String updateAt;
}
```
3. 定义es entity ,注解写明es的索引名和type名
```
@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
@Document(indexName = "index_name", type = "type_name")
public class EntityCollectLabel {

/**
*  查询的时候这个id是查到的es中_id的值，而非定义的id字段，但是插入的时候该id变量的值会插在es中的id字段
*/
    @Id
    private String id;

    @Field(type = FieldType.String)
    private String tagId;

    @Field(type = FieldType.String)
    private String tag;

    @Field(type = FieldType.String)
    private String email;

    @Field(type = FieldType.Date)
    private String createAt;

    @Field(type = FieldType.Date)
    private String updateAt;

}
```

4. es增删改查类
```
package me.handler;

import com.alibaba.fastjson.JSON;
***//省略若干引用
import org.apache.logging.log4j.core.util.JsonUtils;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.search.sort.FieldSortBuilder;
import org.elasticsearch.search.sort.SortOrder;
import org.joda.time.DateTime;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
import org.springframework.data.elasticsearch.core.query.DeleteQuery;
import org.springframework.data.elasticsearch.core.query.NativeSearchQueryBuilder;
import org.springframework.data.elasticsearch.core.query.SearchQuery;
import org.springframework.data.elasticsearch.core.query.UpdateQuery;
import org.springframework.stereotype.Component;
import org.testng.collections.Lists;

import java.util.List;
import java.util.UUID;
import static org.elasticsearch.index.query.QueryBuilders.*;

@Component
@Slf4j
public class CollectLabelHandler extends BaseHandler {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    public void save(CollectLabel collectLabel){
        EntityCollectLabel ecl = new EntityCollectLabel();
        String date = DateTime.now().toString("yyyy-MM-dd HH:mm:ss");
        ecl.setCreateAt(date);
        ecl.setUpdateAt(date);
        ecl.setEmail(collectLabel.getEmail());
        ecl.setTag(collectLabel.getLabel());
        ecl.setTagId(UUID.randomUUID().toString());
        IndexQuery query = new IndexQuery();
        query.setObject(ecl);
        query.setId(ecl.getId);
        query.setIndexName(indexName);
        query.setType(type);
        elasticsearchTemplate.index(query);
    }

    public List<CollectLabel> getByLabel(String label){
        BoolQueryBuilder builder = boolQuery();
        builder.must(termQuery("tag", label));
        //builder.must(nestedQuery("paramContext." + field, termQuery("paramContext." + field + ".trackingId", trackingId)));
        FieldSortBuilder fsb = new FieldSortBuilder("createdAt");
        fsb.order(SortOrder.ASC);

        SearchQuery query = new NativeSearchQueryBuilder()
                .withTypes(TYPE_INTELLIGENT_COLLECT_LABEL.getType())
                .withIndices(EsIndex.INDEX_INTELLIGENT_BASE.getName())
                .withSort(fsb)
                .withQuery(builder).build();
        Iterable<EntityCollectLabel> iterable = elasticsearchTemplate.queryForList(query,EntityCollectLabel.class);
        List<CollectLabel> collectLabels = Lists.newArrayList();
        iterable.forEach(it -> {
            CollectLabel collectLabel = new CollectLabel();
            collectLabel.setCreateAt(it.getCreateAt());
            collectLabel.setUpdateAt(it.getUpdateAt());
            collectLabel.setEmail(it.getEmail());
            collectLabel.setId(it.getId());
            collectLabel.setLabelId(it.getTagId());
            collectLabel.setLabel(it.getTag());
            collectLabels.add(collectLabel);
        });
        return collectLabels;
    }

    public List<CollectLabel> getAll(){
        SearchQuery query = new NativeSearchQueryBuilder().withQuery(matchAllQuery())
                .withTypes(TYPE_INTELLIGENT_COLLECT_LABEL.getType())
                .withIndices(EsIndex.INDEX_INTELLIGENT_BASE.getName())
                .withPageable(new PageRequest(0, 200,
                        new Sort(Sort.Direction.DESC,"createAt"))).build();

        Iterable<EntityCollectLabel> iterable = elasticsearchTemplate.queryForList(query,EntityCollectLabel.class);
        List<CollectLabel> collectLabels = Lists.newArrayList();
        iterable.forEach(it -> {
            CollectLabel collectLabel = new CollectLabel();
            collectLabel.setCreateAt(it.getCreateAt());
            collectLabel.setUpdateAt(it.getUpdateAt());
            collectLabel.setEmail(it.getEmail());
            collectLabel.setLabel(it.getTag());
            collectLabel.setId(it.getId());
            collectLabel.setLabelId(it.getTagId());
            collectLabels.add(collectLabel);
        });
        return collectLabels;
    }

//    public void delete(String labelId) {
//        DeleteQuery collectDeleteQuery = new DeleteQuery();
//        BoolQueryBuilder collectBoolQueryBuilder = boolQuery();
//        collectBoolQueryBuilder.must(termQuery("tagId", labelId));
////        collectBoolQueryBuilder.must(termQuery("trackingId", trackingId));
//        collectDeleteQuery.setQuery(collectBoolQueryBuilder);
//        collectDeleteQuery.setIndex(EsIndex.INDEX_INTELLIGENT_BASE.getName());
//        collectDeleteQuery.setType(TYPE_INTELLIGENT_COLLECT_LABEL.getType());
//        collectDeleteQuery.setPageSize(10);
//        elasticsearchTemplate.delete(collectDeleteQuery);
//    }

    public void delete(String id) {
// 这个id是es中的_id 的值
        elasticsearchTemplate.delete(EntityCollectLabel.class, id);
    }

    public void update(String id, String labelId, String label, String email, String createAt) {
        UpdateQuery updateQuery = new UpdateQuery();
        EntityCollectLabel ecl = new EntityCollectLabel();
        ecl.setCreateAt(createAt);
        ecl.setUpdateAt(DateTime.now().toString("yyyy-MM-dd HH:mm:ss"));
        ecl.setEmail(email);
        ecl.setTag(label);
        ecl.setTagId(labelId);
        ecl.setId(id);

//设置es中_id的值
        updateQuery.setId(ecl.getId());
        updateQuery.setClazz(EntityCollectLabel.class);
//        user.setId(null);
        UpdateRequest request = new UpdateRequest();
        request.doc(JSON.toJSONString(ecl));
        updateQuery.setUpdateRequest(request);
        elasticsearchTemplate.update(updateQuery);

    }
}
```
5. 依赖
```
<dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-elasticsearch</artifactId>
            <version>2.0.2.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch</groupId>
                    <artifactId>elasticsearch</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```


6. 查询大数据量的方法
```
//用上面的查询方法查询大数据量会报这个错误
Caused by: QueryPhaseExecutionException[Result window is too large, from + size must be less than or equal to: [500000] but was [510000]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level parameter.]
  at org.elasticsearch.search.internal.DefaultSearchContext.preProcess(DefaultSearchContext.java:212)

```
*下面是 （正确做法为什么要用这个方法，其中的原理待记录下）*
```
        public List<TrackingDi> readEsByScroll(String gridId){
            BoolQueryBuilder builder = boolQuery();
            builder.must(
//                    QueryBuilders.matchAllQuery()
                    QueryBuilders.termQuery("grid_id", gridId)
            );
            SearchQuery searchQuery = new NativeSearchQueryBuilder()
                    .withQuery(builder)
                    .withIndices(getIndices())
                    .withTypes(EsType.TYPE_DISTRICT.getType())
                    .withPageable(new PageRequest(0,100))
                    .build();
            String scrollId = elasticsearchTemplate.scan(searchQuery,55000,false);
            List<EntityTrackingDi> entityTrackingDiList = new ArrayList<EntityTrackingDi>();
            List<TrackingDi> trackingDiList = new ArrayList<TrackingDi>();
            boolean hasRecords = true;
            while (hasRecords){
                Page<EntityTrackingDi> page = elasticsearchTemplate.scroll(
                    scrollId, 55000L , new SearchResultMapper() {
                        @Override
                        public <T> AggregatedPage<T> mapResults(SearchResponse searchResponse, Class<T> aClass, Pageable pageable) {
                            List<EntityTrackingDi> chunk = new ArrayList<EntityTrackingDi>();
                            for(SearchHit searchHit : searchResponse.getHits()){
                                if(searchResponse.getHits().getHits().length <= 0) {
                                    return null;
                                }
                                EntityTrackingDi entityTrackingDi = new EntityTrackingDi();
                                entityTrackingDi.setWhich_day_week(searchHit.getSource().get("which_day_week").toString());
                                entityTrackingDi.setOrder_count(searchHit.getSource().get("order_count").toString());
                                entityTrackingDi.setShop_business_district_id(searchHit.getSource().get("shop_business_district_id").toString());
                                entityTrackingDi.setCustomer_grid_id(searchHit.getSource().get("customer_grid_id").toString());
                                entityTrackingDi.setGrid_id(searchHit.getSource().get("grid_id").toString());
                                chunk.add(entityTrackingDi);
                            }
                            return new AggregatedPageImpl(chunk);
//                                return new PageImpl<EntityTrackingDi>(chunk);
                        }
                    }
                );
                if(page.hasContent()) {
                    entityTrackingDiList.addAll(page.getContent());
                }
                else{
                    hasRecords = false;
                }
            }
            trackingDiList = TrackingDiConvertor.toTrackingDiList(entityTrackingDiList);
            return trackingDiList;
        }
```
