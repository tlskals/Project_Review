# *JPQL을 이용한 Search 기능 구현*



------



## 왜 JPQL을 사용했는가?



검색 기능은 JPA Repository를 사용하면 쉽게 구현할 수 있습니다.

저희 코드에서는 검색 리스트를 Page로 받아오고 있었고, Controller와 Service가 구현된 상태에서

```java
Page<Place> findByNameIsLike (Pageable pageable, @Param("keyword") String keyword)
```

해당 코드를 사용하면 Parameter로 Keyword를 받아 해당 Keyword가 들어간 데이터를 불러올 수 있었습니다.

하지만 저희 서비스의 모든 검색 기능은 추천순으로 정렬이 되어 있어야 했고

단순 OrderBy를 붙이기엔 Entity 연관 관계에 문제가 있었습니다.

**Place Entity**

```java
@Data
@Entity
public class Place extends BaseTime implements Serializable {
    
...

    @OneToMany(mappedBy = "place", cascade = CascadeType.REMOVE, fetch = FetchType.EAGER)
    private Set<PlaceLikeUser> placeLikeUserList = new HashSet<>();

...
    
}
    
```



저희 서비스에서 추천 기능에는 추천 버튼을 클릭 시, 해당 유저ID를 DB에 조회 후 추천 여부를 Boolean으로 받아

추천/ 추천 취소로 결과를 보내주는 기능을 구현하였습니다.

이러한 기능을 위해서 Place Entity와 PlaceLikeUser Entity를 분리하였고, 일대다 연관 관계로 매핑하였습니다.

이러한 코드 구조로 인하여 전반적인 코드 수정을 하지 않으면 JPA Repository 만으로는 검색 기능을 구현하기 어려웠습니다.



------



## JPQL 작성



먼저 JPQL로 검색 기능을 구현하기 위하여 @Query 애너테이션을 사용하였습니다.

```java
@Query("select p from Place p where p.name like %:keyword%")
Page<Page<Place> searchPlacesByKeyword(Pageable pageable, @Param("keyword") String keyword);>
```

해당 코드는 JPA Repository에서 구현했던 `findByNameIsLike` 와 같은 기능으로 작성하였습니다.



Place는 PlaceLikeUser를 HashSet으로 받아오고 있었고, 추천 개수를 알기 위해서 size 메서드를 사용하기로 했습니다.

또한 추천이 많은 장소부터 적은 장소로 내림차순 정렬을 위해 desc 사용이 필요했습니다.

```java
    @Query("select p from Place p where p.name like %:keyword% order by p.placeLikeUserList.size desc")
    Page<Place> searchPlacesByKeyword(Pageable pageable, @Param("keyword") String keyword);
```



-------



## JPQL 작성을 통해 느낀 점



저희는 지금까지 구현했던 모든 코드에서 JPA Repository를 사용하여 DB 조회를 진행하였기 때문에

해당 기능에 너무 의존하고 있었다는 걸 깨달았습니다.

Spring에서 제공하는 모든 기능에 대하여 해당 기능들을 직접 코드로 구현할 줄 모르는 상황에서

단순 사용하기만 한다면 앞으로도 이런 상황을 겪을 수 있다는 점을 깨달았습니다.
