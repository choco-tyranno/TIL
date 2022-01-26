# The real repository pattern

<img src="https://miro.medium.com/max/1400/1*xxr1Idc8UoNELOzqXcJnag.png"/>

Repository pattern.

+ 안드로이드 공식 문서에서도 가끔 이와 관련한 흔한 실수들이 보임.

### 가장 흔한 5가지 실수 : 

1. 레파지토리가 도메인 모델이 아니라 Dto를 반환.
2. 데이터소스(e.g. ApiServices, Daos ... )가 동일한 Dto를 사용.
3. 앤티티별 레파지토리가 아니라 endpoint 세트별 레파지토리가 있음.
4. 레파지토리가 항상 최신 상태여야 하는 필드를 포함하여, 전체 모델을 캐시함.
5. 하나의 데이터소스가 하나 이상의 레파지토리에서 사용됨.

## 도메인 모델의 필요성

+ 레파지토리 패턴의 핵심.

+ Martin Fowler에 따르면 도메인 모델은 다음과 같다.

  > 도메인 모델 : "행동과 데이터를 모두 통합하는 도메인의 객체 모델."

+ 도메인 모델은 기본적으로 Enterprise-wide한 비즈니스 룰을 나타냄.

+ Domain-Driven Design 작업이나 Layered architectures(Hexagonal, Onion, Clean ...)에 익숙하지 않은 이들을 위한 
  3가지 `도메인 모델 `:

  1. Entity : 식별자(id)가 있고 잠재적으로 변경가능한 일반 개체.
  2. Value object : id 없이 변경 불가능한 개체.
  3. Aggregate root(DDD only) : 다른 엔터티(기본적으로 연결된 개체의 클러스터)와 함께 바인딩되는 엔터티.

+ 단순한 도메인에서 이러한 모델은 데이터베이스 및 네트워크 모델(DTO)와 매우 유사해 보이지만 많은 차이점이 있다.
  차이점 :

  1. 도메인 모델은 데이터와 프로세스가 결합되어 있으며 그 구조가 앱에 가장 적합함.
  2. Dto는 JSON/XML 요청/응답 또는 데이터베이스 테이블의 개체 모델 표현이므로,
     해당 구조는 원격 통신에 가장 적합함.

  예시 : 

  ```kotlin
  // Entity
  data class Product( 
      val id: String,
      val name: String,
      val price: Price,
      val isFavourite: Boolean
  ) {
  // Value object
      data class Price( 
          val nowPrice: Double,
          val wasPrice: Double
      ) {
          companion object {
              val EMPTY = Price(0.0, 0.0)
          }
      }
  }
  ```

  ```kotlin
  // Database DTO
  @Entity(tableName = "Product")
  data class DBProduct(
      @PrimaryKey                
      @ColumnInfo(name = "id")                
      val id: String,                
      @ColumnInfo(name = "name")                
      val name: String,
      @ColumnInfo(name = "nowPrice")
      val nowPrice: Double,
      @ColumnInfo(name = "wasPrice")
      val wasPrice: Double
  )
  ```

  -> 도메인 모델은 프레임워크로부터 자유롭고, 다중 값 속성(위 예시에서 Price와 같이 논리적으로 그룹되는)을 촉진하며, Null Object Pattern을 사용함.
  반면, Dto는 Room이나 Gson 같은 프레임워크와 결합됨.

  이와 같은 분리에 의한 이점 :

+ null체크에서 자유로워지고, 다중값 속성이 전체 모델을 보내도록 강제되지 않음. 개발 용이해짐.

+ 데이터소스에서의 변경이 상위수준 정책에 영향을 미치지 않음.

+ '갓 모델'을 피함으로써 더 많은 관심사 분리가 가능해짐.

+ 안좋은 백엔드 구현이 상위수준 정책에 영향을 주지 않음(안좋은 백엔드 구현의 예 : 단일 정보를 제공하기 위해 2개의 네트워크 요청을 수행해야 하는 경우).



## Data Mapper의 필요성 : 

+ Dto를 도메인 모델에 매핑하거나 그 반대로 매핑.

+ 귀찮은 작업일 수 있으나, 도메인 계층을 무시하고 데이터소스에서부터 UI에 이르기까지 Dto로 이끌어간다면,
   프로덕션에서만 발견될 수 있는 몇 가지 버그가 생성됨(백엔드가 NPE를 생성하는 빈 문자열 대신 null을 전송함).

+ 데이터소스의 동작에서 변경사항으로 인해 버그가 발생하는 것을 피함.

+ Mapper 인터페이스 : 보일러플레이트를 줄임.

  ```kotlin
  interface Mapper<I, O> {
      fun map(input: I): O
  }
  ```

+ List to List mapping : 

  ```kotlin
  // Non-nullable to Non-nullable
  interface ListMapper<I, O>: Mapper<List<I>, List<O>>
  
  class ListMapperImpl<I, O>(
      private val mapper: Mapper<I, O>
  ) : ListMapper<I, O> {
      override fun map(input: List<I>): List<O> {
          return input.map { mapper.map(it) }
      }
  }
  ```

  ```kotlin
  // Nullable to Non-nullable
  interface NullableInputListMapper<I, O>: Mapper<List<I>?, List<O>>
  
  class NullableInputListMapperImpl<I, O>(
      private val mapper: Mapper<I, O>
  ) : NullableInputListMapper<I, O> {
      override fun map(input: List<I>?): List<O> {
          return input?.map { mapper.map(it) }.orEmpty()
      }
  }
  ```

  ```kotlin
  // Non-nullable to Nullable
  interface NullableOutputListMapper<I, O>: Mapper<List<I>, List<O>?>
  
  class NullableOutputListMapperImpl<I, O>(
      private val mapper: Mapper<I, O>
  ) : NullableOutputListMapper<I, O> {
      override fun map(input: List<I>): List<O>? {
          return if (input.isEmpty()) null else input.map { mapper.map(it) }
      }
  }
  ```

## 각 데이터소스별 다른 모델의 필요성 :

+  싱글 모델로 사용할 경우,  다음 리스크들을 생각해볼 수 있음.

  1. 필요 이상으로 캐싱.
  2. Response에 필드를 추가하려면 데이터베이스 마이그레이션이 필요함(@Ignore 주석을 사용하지 않는 한).
  3. Request body에 보내지 말아야할 캐시하는 모든 필드에 @Transient 주석이 필요함.
  4. 새 필드를 생성하지 않는 한 동일한 이름의 필드는 동일한 데이터 유형이어야함(예 : 네트워크 응답으로부터 price를 string으로 parse하고 price를 Int로 캐시할 수 없음).

  -> 별도의 모델을 사용하는 것이 더 적은 유지 관리를 만듬.

## 필요한 것만 캐싱하는 것의 필요성 :

+ 예 : 원격 카탈로그에서 제품 목록을 받아와서 로컬 위시리스트에 있는 제품인 경우 하트를 표시하도록 하는 케이스.
  -> 원격 모델과 똑같은 데이터베이스 모델이 필요하지 않음. SharedPreference로 충분히 가능하기때문. 

```kotlin
class ProductRepositoryImpl(
    private val productApiService: ProductApiService,
    private val productDataMapper: Mapper<DataProduct, Product>,
    private val productPreferences: ProductPreferences
) : ProductRepository {

    override fun getProducts(): Single<Result<List<Product>>> {
        return productApiService.getProducts().map {
            when(it) {
                is Result.Success -> Result.Success(mapProducts(it.value))
                is Result.Failure -> Result.Failure<List<Product>>(it.throwable)
            }
        }
    }

    private fun mapProducts(networkProductList: List<NetworkProduct>): List<Product> {
        return networkProductList.map { 
            productDataMapper.map(DataProduct(it, productPreferences.isFavourite(it.id)))
        }
    }
      
}
```

```kotlin
// A wrapper for handling failing requests
sealed class Result<T> {

    data class Success<T>(val value: T) : Result<T>()

    data class Failure<T>(val throwable: Throwable) : Result<T>()

}

// A DataSource for the SharedPreferences
interface ProductPreferences {

    fun isFavourite(id: String?): Boolean

}

// A DataSource for the Remote DB
interface ProductApiService {

    fun getProducts(): Single<Result<List<NetworkProduct>>>

    fun getWishlist(productIds: List<String>): Single<Result<List<NetworkProduct>>>
    
}

// A cluster of DTOs to be mapped into a Product
data class DataProduct(
    val networkProduct: NetworkProduct,
    val isFavourite: Boolean
)
```

```kotlin
class ProductRepositoryImpl(
    private val productApiService: ProductApiService,
    private val productDataMapper: Mapper<DataProduct, Product>,
    private val productPreferences: ProductPreferences
) : ProductRepository {
  
    override fun getWishlist(): Single<Result<List<Product>>> {
        return productApiService.getWishlist(productPreferences.getFavourites()).map {
            when (it) {
                is Result.Success -> Result.Success(mapWishlist(it.value))
                is Result.Failure -> Result.Failure<List<Product>>(it.throwable)
            }
        }
    }

    private fun mapWishlist(wishlist: List<NetworkProduct>): List<Product> {
        return wishlist.map {
            productDataMapper.map(DataProduct(it, true))
        }
    }
      
}
```

