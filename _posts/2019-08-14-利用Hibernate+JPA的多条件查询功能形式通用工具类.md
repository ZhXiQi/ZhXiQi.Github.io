---
layout: post
title: 利用Hibernate+JPA的多条件查询功能形式通用工具类
data: 2019-08-14
tag: Specification、jpa、hibernate、pageable
---



## 利用Hibernate+JPA的多条件查询功能形式通用工具类

Hibernate+JPA框架已经帮我们封装好了一套多条件查询功能，但是在工作的时候发现还是有不少小伙伴还是使用dao手动拼接SQL的方式来进行多条件动态查询，这样导致查询语句看起来很不雅观，所以为了更加雅观，增加代码可读性，我将这个多条件及分页查询写成了一个通用的工具类方法（会不会有歧义？工具类涉及业务？但是也是通用的），talk is cheap，show me the code！！！！



### 1.主角多条件查询工具类

```java
package cn.xxxx.util;

import cn.xxxx.model.vo.QueryVO;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.util.ObjectUtils;

import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;
import java.util.ArrayList;
import java.util.List;

/**
 * @author ZhXiQi
 * @Title: 工具类支持泛型，使用的时候传入实体，便可以生成支持对应实体的Specification，达到通用效果
 * @date 2019-08-09 15:26
 */
public class BusinessUtil<T> {
    /**
     * 由于是作为工具类的方式，所以next等几个方法还是用到了Pageable接口的实现类
     * @param pageNum 页码，传参的时候就需要处理好是否减一
     * @param pageSize 页数
     * @param asc 是否递增，基本类型
     * @param field 按哪个字段进行排序
     * @return
     */
    public Pageable getPageable(int pageNum, int pageSize, boolean asc, String... field){
        Pageable pageable = new Pageable() {
            @Override
            public int getPageNumber() {
                return pageNum;
            }

            @Override
            public int getPageSize() {
                return pageSize;
            }

            @Override
            public long getOffset() {
                return pageNum * pageSize;
            }

            @Override
            public Sort getSort() {
                if (asc){
                    return new Sort(Sort.Direction.ASC,field);
                }else {
                    return new Sort(Sort.Direction.DESC,field);
                }
            }

            @Override
            public Pageable next() {
                return PageRequest.of(this.getPageNumber() + 1,this.getPageSize(), this.getSort());
            }

            @Override
            public Pageable previousOrFirst() {
                return PageRequest.of(this.getPageNumber() == 0 ? 0:this.getPageNumber() - 1,this.getPageSize(),this.getSort());
            }

            @Override
            public Pageable first() {
                return PageRequest.of(0,this.getPageSize(),this.getSort());
            }

            @Override
            public boolean hasPrevious() {
                return pageNum > 0;
            }
        };
        return pageable;
    }

    /**
     * 
     * @param params 参数列表
     * @return
     */
    public Specification<T> getSpec(List<QueryVO> params){
        Specification specification = new Specification() {
            @Override
            public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {
                List<Predicate> predicateList = new ArrayList<>();
                params.parallelStream().forEach(vo -> {
                    String key = vo.getKey();
                    Object value = vo.getValue();
                    if (!ObjectUtils.isEmpty(value)) {
                        switch (vo.getQueryEnum()) {
                            case equal:
                                predicateList.add(criteriaBuilder.equal(root.get(key), value));
                                break;
                            case in:
                                CriteriaBuilder.In<Object> objectIn = criteriaBuilder.in(root.get(key));
                                if (value instanceof List) {
                                    ((ArrayList<Integer>) value).parallelStream().forEach(v -> objectIn.value(v));
                                    predicateList.add(objectIn);
                                }
                                break;
                            case like:
                                predicateList.add(criteriaBuilder.like(root.get(key).as(String.class),"%"+value+"%"));
                                break;
                            case ge:
                                if (value instanceof Number){
                                    predicateList.add(criteriaBuilder.ge(root.get(key), (Number) value));
                                }
                                break;
                            case gt:
                                if (value instanceof Number){
                                    predicateList.add(criteriaBuilder.gt(root.get(key), (Number) value));
                                }
                                break;
                            case le:
                                if (value instanceof Number){
                                    predicateList.add(criteriaBuilder.le(root.get(key), (Number) value));
                                }
                                break;
                            case lt:
                                if (value instanceof Number){
                                    predicateList.add(criteriaBuilder.lt(root.get(key), (Number) value));
                                }
                                break;
                            case nq:
                                predicateList.add(criteriaBuilder.notEqual(root.get(key),value));
                                break;
                            default:
                                break;
                        }
                    }
                });
                return criteriaBuilder.and(predicateList.toArray(new Predicate[predicateList.size()]));
            }
        };
        return specification;
    }
}
```



### 2.封装查询条件的VO——QueryVo

```java
package cn.xxx.model.vo;

import cn.xxx.business.constant.QueryEnum;
import lombok.Builder;
import lombok.Data;

/**
 * @author ZhXiQi
 * @Title:
 * @date 2019-08-09 17:55
 */
@Builder
@Data
public class QueryVO {
		/**
     * 多条件查询的条件字段，对应为Hibernate实体类中的字段，而不是数据库里的
     */
    private String key;

  	/**
     * 查询类型，如 equal、like 等
     */
    private QueryEnum queryEnum;

  	/**
     * 查询条件值
     */
    private Object value;
}

```



### 3.多条件查询枚举——QueryEnum

```java
package cn.xxx.business.constant;

/**
 * @author ZhXiQi
 * @Title:
 * @date 2019-08-09 16:24
 */
public enum QueryEnum {
    equal,
    like,
    in,
    nq,
    gt,
    ge,
    lt,
    le
}

```

### 4.使用

```java
//查询条件列表
List<QueryVO> params = new ArrayList<>();
        
//查询条件封装，builder模式更加清晰
params.add(QueryVO.builder().key("userId").queryEnum(QueryEnum.equal).value(userInfo.getUuid()).build());
        params.add(QueryVO.builder().key("status").queryEnum(QueryEnum.in).value(status).build());
BusinessUtil<TestEntity> testEntityBusinessUtil = new BusinessUtil<>();
Page<TestEntity> testEntityPage = testEntityRepo.findAll(testEntityBusinessUtil.getSpec(params), 
//支持多个字段排序
                                                         testEntityBusinessUtil.getPageable(page - 1, pageSize, sort == null ? false : sort, "createTime"));
List<TestEntity> list = testEntityPage.getContent();
```



