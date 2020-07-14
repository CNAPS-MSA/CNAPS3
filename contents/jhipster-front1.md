# Vue.js로 client 변경하기

Jhipster로 gateway개발 시 client를 선택할 수 있는데, 기본적으로는 Angular와 react만 제공된다. Vue.js로 개발하기 위해서는 Jhipster 프로젝트를 만들 때 아래와 같이 command를 입력해야한다.

```bash
mkdir gateway
cd gateway
jhipster --blueprint vuejs
```

이후 Microservice gateway로 app종류를 설정하고 적절하게 옵션선택 후, 기존 gateway에 작성했던 user service 관련 내용을 복구시킨 뒤 개발을 시작한다.

## NavBar에 메뉴 추가 

1. jhi-navbar.vue에 rent 추가하기
   
   `src -> main -> webapp -> app -> core -> jhi-navbar` 의 경로에 있는 파일들이 JHipster 페이지의 상단 헤더 부분임.

   navbar 폴더 내부의 jhi-navbar.vue 를 열어 아래 코드를 추가.

   ```html
   <!-- NEW MENU-->
             <b-nav-item to="/rent">
                    <font-awesome-icon icon="book"/>
                    <span v-text="$t('global.menu.rentalpage')">Rental Page</span>
             </b-nav-item>
    ```

    코드를 살펴보면, **to="/rent"**라는 옵션이 보입니다. 이부분이 바로 해당 메뉴를 클릭하면 원하는 페이지로 연결시켜주는 부분이다.

    그리고 fontawesome icon의 'book' 아이콘을 사용할 예정이다. fontawesome icon은 `... app -> shared-> config-> config.ts`에 아래 코드와 같이 import 후, library에 추가하면 사용할 수 있다.

    ```js
    
    import {
        faBook
    } from "@fortawesome/free-solid-svg-icons/faBook";

    export function initFortAwesome(vue) {
  library.add( ...

    ,faBook);
    }

    ```


### Vue.js router에 등록

위에서 설명한 navbar에 `to="rent"`라는 코드를 통해 저 메뉴를 누르면 `"rent"`로 이동하게 된다.
그럼 이 `"rent"`를 Vue.js는 어떻게 찾을까?

바로, router를 통해 찾게된다.

`... app -> router -> index.ts` 로 이동한다.

파일을 열어보면 아주 많은 path들이 등록되어있는 것을 볼 수 있다.

여기에 아래와 같이 `"rent"`를 위한 path를 등록해준다. 

```js
    ...
    },
    {
      path: '/rent',
      name: 'BookRental',
      component: BookRental,
      meta: { authorities: [Authority.USER]}
    }

```

코드를 살펴보면, path와 name, component, meta가 있습니다. 유추해볼 수 있듯이 path는 vue파일에서 to로 연결됩니다. name은 vue파일에서 to대신 name으로 연결 시에 사용된다.
component는 vue 파일에서 쓰이는 component파일 이름. 

하지만 component 파일 이름을 vue는 어떻게 찾을 수 있을까?

바로 아래와 같이 해당 component를 import 하여 찾게된다.

```js
const BookRental = () => import('../cnaps/book-rental-service/book-rental.vue');
```
위 코드는 index.ts파일의 `Vue.use(Router);` 라는 코드 윗부분에 추가. 

### 한국어 번역 등록하기

자, 다시 navbar에 추가한 코드를 살펴보면,

```html
    <b-nav-item to="/rent">
        <font-awesome-icon icon="book"/>
            <span v-text="$t('global.menu.rentalpage')">Rental Page</span>
    </b-nav-item>
``` 

span부분에 `v-text="$t('global.menu.rentalpage')"`는 바로 이 영문 text를 해당 언어로 바꿔쓴다는 것!

webapp -> i18n -> ko -> global.json 파일로 이동.

```json
"menu": {
      "home": "홈",
      "rentalpage": "도서 대여",
      "jhipster-needle-menu-add-element": "JHipster will add additional menu entries here (do not translate!)",
      "entities": {
        "main": "Entities",
        "bookBook": "Book",
        "bookInStockBook": "In Stock Book",
        "bookCatalogBookCatalog": "Book Catalog",
        "bookCatalogTopListBooks": "Top List Books",
        "rentalRental": "Rental",
        "rentalOverdueItem": "Overdue Item",
        "rentalRentedItem": "Rented Item",
        "rentalReturnedItem": "Returned Item",
        "jhipster-needle-menu-add-entry": "JHipster will add additional entities here (do not translate!)"
      }
```

rentalpage를 menu의 바로 하위에 등록하였기 때문에, 위 코드처럼 home과 같은 위치에 **rentalpage를 도서 대여** 로 번역을 등록함.

### 결과 확인

이제 gateway를 실행시켜보면, 아래와 같은 화면을 볼 수 있다.

<img width="940" alt="image" src="https://user-images.githubusercontent.com/18453570/87122439-d9fd0000-c2bf-11ea-92f6-3bad80b43821.png">


## book-rental 기능을 위한 모듈 개발

router의 index.ts에 추가한 내용처럼 

```js
    const BookRental = () => import('../cnaps/book-rental-service/book-rental.vue');
```

webApp -> app 에 cnaps라는 폴더를 생성한다. 이제 이 cnaps 라는 폴더 하위에 각 기능들을 위한 폴더를 생성하고 해당 폴더들 안에 각 기능을 위한 모듈을 개발한다.

Vue.js에서는 기본적으로 component.ts, service.ts, vue파일이 세트로 묶인다. 여기에 각 페이지별로 뻗어가는 하위 기능을 위한 view는 component와 vue파일만 새로 생성한다.

이제 book-rental이란 모듈을 개발할 예정인데, 이 모듈 내에는 우선적으로 도서 조회, 검색, 도서 세부정보 보기를 만들 것이다. 추후 대여/반납 기능을 추가할 예정이다. 

따라서 아래 이미지와 같이 폴더 및 패키지를 생성한다.

![image](https://user-images.githubusercontent.com/18453570/87122484-f7ca6500-c2bf-11ea-9abf-dac28ea8bba2.png)

우선 Vue.js의 component, service, vue파일은 jhipster에서 제공하는 entities의 기존 내용을 그대로 가져와 수정 및 추가하는 방향으로 개발하였다.

### book-rental.vue작성

book-rental.vue의 전체 코드는 아래와 같다. (조회와 검색 기능만 있음)

```html
<template>
    <div>
        <h2 id="page-heading">
            <span v-text="$t('global.menu.rentalpage')">Rental Page</span>
        </h2>
        <b-alert :show="dismissCountDown"
            dismissible
            :variant="alertType"
            @dismissed="dismissCountDown=0"
            @dismiss-count-down="countDownChanged">
            {{alertMessage}}
        </b-alert>
        <br/>
        <div class="alert alert-warning" v-if="!isFetching && books && books.length === 0">
            <span>No books found</span>
        </div>
        <div class="input-group mb-3">
            <label>
                <input type="text" class="form-control" placeholder="Search by title"
                       v-model="title"/>
            </label>
            <div class="input-group-append">
                <button class="btn btn-outline-secondary" type="button"
                        @click="search(title)"
                >
                    Search
                </button>
            </div>
        </div>
        <div class="table-responsive" v-if="books && books.length > 0">
            <table class="table table-striped">
                <thead>
                <tr>
                    <th><span v-text="$t('gatewayApp.bookCatalogBookCatalog.title')">Title</span></th>
                    <th><span v-text="$t('gatewayApp.bookCatalogBookCatalog.description')">Description</span></th>
                    <th><span v-text="$t('gatewayApp.bookCatalogBookCatalog.classification')">Classification</span></th>
                    <th><span v-text="$t('gatewayApp.bookCatalogBookCatalog.author')">Author</span></th>
                    <th><span v-text="$t('gatewayApp.bookCatalogBookCatalog.publicationDate')">Publication Date</span></th>
                    <th><span v-text="$t('gatewayApp.bookCatalogBookCatalog.rented')">Rented</span></th>
                    <th><span v-text="$t('gatewayApp.bookCatalogBookCatalog.rentCnt')">Rental Count</span></th>
                    <th></th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="book in books"
                    :key="book.title">
                    <td>
                        <router-link :to="{name: 'BookRentalView', params: {bookTitle: book.title}}">{{book.title}}</router-link>
                    </td>
                    <td>{{book.description}}</td>
                    <td>{{book.classification}}</td>
                    <td>{{book.author}}</td>
                    <td>{{book.publicationDate}}</td>
                    <td>{{book.rented}}</td>
                    <td>{{book.rentCnt}}</td>
                    <td class="text-right">
                        <div class="btn-group">
                            <router-link :to="{name: 'BookRentalView', params: {bookTitle: book.title}}" tag="button" class="btn btn-info btn-sm details">
                                <font-awesome-icon icon="eye"></font-awesome-icon>
                                <span class="d-none d-md-inline" v-text="$t('entity.action.view')">View</span>
                            </router-link>

                        </div>
                    </td>
                </tr>
                </tbody>
            </table>
        </div>
        <div v-show="books && books.length > 0">
            <div class="row justify-content-center">
                <jhi-item-count :page="page" :total="queryCount" :itemsPerPage="itemsPerPage"></jhi-item-count>
            </div>
            <div class="row justify-content-center">
                <b-pagination size="md" :total-rows="totalItems" v-model="page" :per-page="itemsPerPage" :change="loadPage(page)"></b-pagination>
            </div>
        </div>
    </div>
</template>

<script lang="ts" src="./book-rental.component.ts">
</script>
```

윗부분부터 살펴본다.

1. 페이지 이름 수정 
   
    ```html
    <h2 id="page-heading">
        <span v-text="$t('global.menu.rentalpage')">Rental Page</span>
    </h2>
    ```
    먼저, 페이지 이름을 메뉴에 등록한 이름으로 변경한다. 

2. 책이 없는 경우 경고 표시 & 검색어 입력 및 버튼 만들기
   
    ```html
        <div class="alert alert-warning" v-if="!isFetching && books && books.length === 0">
            <span>No books found</span>
        </div>
        <div class="input-group mb-3">
            <label>
                <input type="text" class="form-control" placeholder="Search by title"
                       v-model="title"/>
            </label>
            <div class="input-group-append">
                <button class="btn btn-outline-secondary" type="button"
                        @click="search(title)"
                >
                    Search
                </button>
            </div>
        </div>
    ```

    `v-if`는 말 그래도 if문이다. 만약 book catalog에 등록된 책이 없으면 No books found라는 메세지가 뜨도록 하였다. 여기서 book catalog list가 바로 `books`라는 변수로 쓰였다.

    그 밑에는 입력 폼을 추가하였다. 여기서 v-model은 입력된 내용을 `title`이라는 변수에 넣는다는 것이며, 버튼을 누르면 `@click="search(title)"`이 실행되어 search(title)이라는 메소드가 실행된다.

    그럼 이 `books`라는 변수와 `search(title)`라는 메소드는 어디에 선언한 것일까?

    바로 component.ts파일에 선언된다. vue 파일의 전체 소스코드 맨 하단을 보면 `<script>`로 묶인 부분에 component.ts파일이 source라는 것이 명시되어있다.

### book-rental-component.ts파일 수정
   
```js
        import { mixins } from 'vue-class-component';

        import { Component, Vue, Inject } from 'vue-property-decorator';
        import Vue2Filters from 'vue2-filters';
        import { IRental } from '@/shared/model/rental/rental.model';
        import { IBookCatalog } from '@/shared/model/bookCatalog/book-catalog.model';
        import AlertMixin from '@/shared/alert/alert.mixin';

        import BookRentalService from './book-rental.service';

        @Component({
        mixins: [Vue2Filters.mixin],
        })
        export default class BookRental extends mixins(AlertMixin) {
        @Inject('bookRentalService') private bookRentalService: () => BookRentalService;
        private removeId: number = null;
        public itemsPerPage = 20;
        public queryCount: number = null;
        public page = 1;
        public previousPage = 1;
        public propOrder = 'id';
        public reverse = false;
        public totalItems = 0;
        public title = '';
        public rentals: IRental[] = [];
        public books: IBookCatalog[] = [];
        public isFetching = false;

        public mounted(): void {
            this.retrieveAllBooks();
        }

        public clear(): void {
            this.page = 1;
            this.retrieveAllBooks();
        }

        public retrieveAllBooks(): void {
            this.isFetching = true;

            const paginationQuery = {
            page: this.page - 1,
            size: this.itemsPerPage,
            sort: this.sort(),
            };
            this.bookRentalService()
            .retrieve(paginationQuery)
            .then(
                res => {
                this.books = res.data;
                this.totalItems = Number(res.headers['x-total-count']);
                this.queryCount = this.totalItems;
                this.isFetching = false;
                },
                err => {
                this.isFetching = false;
                }
            );
        }
        public sort(): Array<any> {
            const result = [this.propOrder + ',' + (this.reverse ? 'asc' : 'desc')];
            if (this.propOrder !== 'id') {
            result.push('id');
            }
            return result;
        }

        public loadPage(page: number): void {
            if (page !== this.previousPage) {
            this.previousPage = page;
            this.transition();
            }
        }

        public transition(): void {
            this.retrieveAllBooks();
        }

        public changeOrder(propOrder): void {
            this.propOrder = propOrder;
            this.reverse = !this.reverse;
            this.transition();
        }

        public closeDialog(): void {
            (<any>this.$refs.removeEntity).hide();
        }
        
        public search(title: String): void {
            let foundBook: IBookCatalog[] = [];
            this.bookRentalService()
            .findByTitle(title)
            .then(res => {
                foundBook.push(res);
                this.books = foundBook;
            });
        }
        }

```

위 소스 코드를 차근 차근 살펴보자.
먼저 윗부분부터 살펴보면 `@Inject('bookRentalService') private bookRentalService: () => BookRentalService;`라는 코드로 bookRentalService를 주입시킨다.
이 bookRentalService는 추후 설명할 예정이지만, 간단히 설명하자면 다른 마이크로 서비스의 REST API controller로 요청을 주고받는 곳이다.
        
```js
    public title = '';
    public rentals: IRental[] = [];
    public books: IBookCatalog[] = [];
```
위처럼 vue파일에 쓰일 변수들을 초기화 해준다. title은 검색할 때 쓰이는 v-model 변수로 설명했다.
rentals와 books는 Jhipster에서 rental과 bookCatalog 서비스와 연결한 후 생성한 client model과 연결시킨다. 해당 모델들은 `webApp -> app -> shared -> model`에서 확인할 수 있다.

그 밑의 메소드들이 바로 vue의 템플릿에서 사용되는 메소드들이다. 

```js
    public search(title: String): void {
            let foundBook: IBookCatalog[] = [];
            this.bookRentalService()
            .findByTitle(title)
            .then(res => {
                foundBook.push(res);
                this.books = foundBook;
            });
        }
        }
```

    이부분이 바로 vue파일에서 선언한 search이다. 현재는 title이 정확하게 일치하는 도서만 검색되게 하였기 때문에 위처럼 작성하였고, 추후 수정할 예정이다.
    
    위 코드에서 보면 bookRentalService를 호출하고 있는데, 이부분은 잠시 후 설명한다.

### book-rental.vue
    
다시, book-rental.vue파일 코드를 살펴보자.

```html
    <tr v-for="book in books"
                    :key="book.title">
                    <td>
                        <router-link :to="{name: 'BookRentalView', params: {bookTitle: book.title}}">{{book.title}}</router-link>
                    </td>
                    <td>{{book.description}}</td>
                    <td>{{book.classification}}</td>
                    <td>{{book.author}}</td>
                    <td>{{book.publicationDate}}</td>
                    <td>{{book.rented}}</td>
                    <td>{{book.rentCnt}}</td>
                    <td class="text-right">
                        <div class="btn-group">
                            <router-link :to="{name: 'BookRentalView', params: {bookTitle: book.title}}" tag="button" class="btn btn-info btn-sm details">
                                <font-awesome-icon icon="eye"></font-awesome-icon>
                                <span class="d-none d-md-inline" v-text="$t('entity.action.view')">View</span>
                            </router-link>

                        </div>
                    </td>
                </tr>
```    
우선 v-for은 for문으로 book catalog에서 가져온 도서들을 하나씩 돌아가며 하단 코드를 실행시킨다. 이때 key로 book.title인것을 볼 수 있는데, index를 의미한다. (book.id로 수정할 예정이다.)

첫번째로 눈에 띄는 것은 `<router-link>`이다. 이것은 위에서 설명한대로 router의 index.ts에 추가하였던 path의 name과 연결되는 곳이다. 
    
첫번째 router-link를 보면 'BookRentalView'에 연결되며, 전달되는 param은 bookTitle로 book.title이 입력됨을 알 수 있다. 두번째 router-link도 마찬가지이다.

이부분을 통해 바로 **vue에서 다른 vue파일로 param을 넘기는 방식**을 확인할 수 있다.

BookRentalView는 BookRentalService를 설명한 뒤 간략하게 설명하도록하겠다.

### book-rental-service.ts

앞서 component에서 bookRentalService를 주입시켰다. 이렇게 bookRentalService를 주입시키기 위해선 이 애플리케이션에 bookRentalService를 선언해야한다.

서비스를 선언하는 부분은 `webApp -> app -> main.ts`에서 선언할 수 있다.

```js

import BookRentalService from '@/cnaps/book-rental-service/book-rental.service';

...

new Vue({
  el: '#app',
  components: { App },
  template: '<App/>',
  router,
  provide: {
    ...
    bookRentalService: () => new BookRentalService(),
  },
  i18n,
  store,
});

```

이제 book-rental-service.ts를 살펴보자.

```js
import axios from 'axios';

import buildPaginationQueryOpts from '@/shared/sort/sorts';

import { IRental } from '@/shared/model/rental/rental.model';
import { IBookCatalog } from '@/shared/model/bookCatalog/book-catalog.model';

const rentalApiUrl = 'services/rental/api/rentals';
const bookApiUrl = 'services/bookcatalog/api/book-catalogs';

export default class BookRentalService {
  public find(id: number): Promise<IBookCatalog> {
    return new Promise<IBookCatalog>((resolve, reject) => {
      axios
        .get(`${bookApiUrl}/${id}`)
        .then(res => {
          resolve(res.data);
        })
        .catch(err => {
          reject(err);
        });
    });
  }

  public retrieve(paginationQuery?: any): Promise<any> {
    return new Promise<any>((resolve, reject) => {
      axios
        .get(bookApiUrl + `?${buildPaginationQueryOpts(paginationQuery)}`)
        .then(res => {
          resolve(res);
        })
        .catch(err => {
          reject(err);
        });
    });
  }

  public findByTitle(title: String): Promise<IBookCatalog> {
    return new Promise<IBookCatalog>((resolve, reject) => {
      axios
        .get(`${bookApiUrl}/title/${title}`)
        .then(res => {
          resolve(res.data);
        })
        .catch(err => {
          reject(err);
        });
    });
  }
  
}
```
book-catalog-service.ts의 코드 또한 매우 직관적이다. 

```js
const rentalApiUrl = 'services/rental/api/rentals';
const bookApiUrl = 'services/bookcatalog/api/book-catalogs';
```
이부분은 url을 선언하는 부분이다. 이 url을 통해 gateway에 등록된 마이크로 서비스의 REST API로 요청을 주고 받는다.

앞서 component에서 생성한 search 메소드를 살펴보면

```js
    public search(title: String): void {
            let foundBook: IBookCatalog[] = [];
            this.bookRentalService()
            .findByTitle(title)
            .then(res => {
                foundBook.push(res);
                this.books = foundBook;
            });
        }
        }
```
`this.bookRentalService().findBytitle(title)`로 검색을 위해 입력한 title을 bookRentalService의 findByTitle메소드로 보낸다.
서비스에서 findByTitle()을 실행시키면 axios로 요청을 보내고 data를 받아 resolve한다. 이 data를 다시 컴포넌트의 search 메소드가 받고, book-rental에서 사용하는 books Array에 넣어준다.

### BookRentalView

BookRentalView는 도서의 상세 정보를 조회했을 때 나오는 페이지로, book-rental-details에 해당한다. 이것 또한 새로운 페이지로 이동하는 것이기 때문에 우선 router에 등록해주어야한다.

router의 index.ts에 아래와 같은 코드를 추가해준다.

```js
{
      path: '/rent/:bookTitle/view',
      name: 'BookRentalView',
      component: BookRentalDetails,
      meta: { authorities: [Authority.USER]}
    }
```

path를 보면 `/rent/:bookTitle/view`로 중간에 bookTitle이 있는 것을 확인할 수 있다. 이는 param을 bookTitle로 가져온다는 것이다. (추후 bookId로 수정될 수 있다.)
또한 BookRentalView라는 이름으로 vue파일에서 연결될 것이다. book-rental.vue 코드에서 아래와 같이 확인할 수 있다.

```html
<td>
    <router-link :to="{name: 'BookRentalView', params: {bookTitle: book.title}}">{{book.title}}</router-link>
</td>
```

component 는 BookRentalDetails란 이름으로 아래와 같이 등록해준다.

```js
const BookRentalDetails = () => import('../cnaps/book-rental-service/book-rental-details.vue');
```

component는 path와 마찬가지로 index.ts라는 동일한 파일 상단에 등록해준다.

이제 book-rental-details-component를 확인해보자.

### book-rental-details-component.ts작성

```js
import { Component, Inject, Vue } from 'vue-property-decorator';
import { IBookCatalog } from '@/shared/model/bookCatalog/book-catalog.model';
import BookRentalService from '@/cnaps/book-rental-service/book-rental.service';
@Component
export default class BookRentalDetails extends Vue {
  @Inject('bookRentalService') private bookRentalService: () => BookRentalService;
  public book: IBookCatalog = {};

  beforeRouteEnter(to, from, next) {
    next(vm => {
      if (to.params.bookTitle) {
        vm.retrieveBookRental(to.params.bookTitle);
      }
    });
  }

  public retrieveBookRental(bookTitle) {
    this.bookRentalService()
      .findByTitle(bookTitle)
      .then(res => {
        this.book = res;
      });
  }

  public previousState() {
    this.$router.go(-1);
  }
}
```

- bookRentalService 주입 : book-rental-details 또한 BookRentalService 의 주입이 필요하다. Book Rental과 연결되어있는 기능이기 때문이다.
- book : IBookCatalog= {} : Book Catalog의 front model을 가져와 선언한다. book-rental-details.vue에서 `book`이라는 명칭을 사용해 bookCatalog를 가져오게된다.
- beforeRouteEnter : vue의 기능 중 하나인 **네비게이션 가드**로, Vue Router로 특정 URL에 접근할 때 해당 URL의 접근을 막는 방법을 의미한다. 주로 사용자 인증정보가 없으면 페이지에 접근을 못하게 하는데에 주로 쓰인다.
  - 여기서는 다른 의미로 쓰였는데, 상세정보 조회 버튼을 눌렀을 때 해당 책에 대한 key정보(bookTitle)이 param으로 넘어오지 않으면 페이지 이동이 제한된다.
- retrieveBookRental : beforeRouteEnter 함수의 코드를 보면 조건 통과 후 retrieveBookRental을 호출한다. retrieveBookRental은 넘겨받은 bookTitle을 bookRentalService의 findByTitle을 실행시켜 book 정보를 가져온다.

### book-rental-details.vue

```html
<template>
    <div class="row justify-content-center">
        <div class="col-8">
            <div v-if="book">
                <h2 class="jh-entity-heading"><span v-text="$t('gatewayApp.bookCatalogBookCatalog.detail.header')">Book Details</span></h2>
                <dl class="row jh-entity-details">
                    <dt>
                        <span v-text="$t('gatewayApp.bookCatalogBookCatalog.title')">Title</span>
                    </dt>
                    <dd>
                        <span>{{book.title}}</span>
                    </dd>
                    <dt>
                        <span v-text="$t('gatewayApp.bookCatalogBookCatalog.description')">Description</span>
                    </dt>
                    <dd>
                        <span>{{book.description}}</span>
                    </dd>
                    <dt>
                        <span v-text="$t('gatewayApp.bookCatalogBookCatalog.author')">Author</span>
                    </dt>
                    <dd>
                        <span>{{book.author}}</span>
                    </dd>
                    <dt>
                        <span v-text="$t('gatewayApp.bookCatalogBookCatalog.publicationDate')">Publication Date</span>
                    </dt>
                    <dd>
                        <span>{{book.publicationDate}}</span>
                    </dd>
                    <dt>
                        <span v-text="$t('gatewayApp.bookCatalogBookCatalog.classification')">Classification</span>
                    </dt>
                    <dd>
                        <span>{{book.classification}}</span>
                    </dd>
                    <dt>
                        <span v-text="$t('gatewayApp.bookCatalogBookCatalog.rented')">Rented</span>
                    </dt>
                    <dd>
                        <span>{{book.rented}}</span>
                    </dd>
                    <dt>
                        <span v-text="$t('gatewayApp.bookCatalogBookCatalog.rentCnt')">Rent Cnt</span>
                    </dt>
                    <dd>
                        <span>{{book.rentCnt}}</span>
                    </dd>
                </dl>
            </div>
        </div>
    </div>
</template>

<script lang="ts" src="./book-rental-details.component.ts">
</script>
```

bookRentalDetailsView에 연결된 vue 파일이다. 상세 정보를 보여주는 만큼 심플하다. (주 코드 설명은 book-rental.vue에서 설명하였으므로 생략)

## Vue.js 개발 간단 요약

Vue.js에서 어떠한 페이지를 생성해 기능을 부여하는 개발 순서는 아래와 같다. 

1. 개발하고자 하는 모듈의 package(폴더)생성
2. 모듈 폴더 내에 해당 기능을 위한 name-component.ts, name-service.ts, name.vue파일 생성 및 소스 개발
   1. component는 vue에서 사용되는 메소드 및 변수 선언
   2. service는 Microservices와의 REST API 통신을 위한 메소드 및 변수 선언
3. component 및 service 를 전역 등록
4. Path를 router에 등록
5. navBar 또는 Home에서 이동할 수 있는 메뉴 추가

