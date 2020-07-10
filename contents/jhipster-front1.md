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

![](/images/judy/2020-07-10-14-11-39.png)


## book-rental 기능을 위한 모듈 개발

router의 index.ts에 추가한 내용처럼 

```js
    const BookRental = () => import('../cnaps/book-rental-service/book-rental.vue');
```

webApp -> app 에 cnaps라는 폴더를 생성한다. 이제 이 cnaps 라는 폴더 하위에 각 기능들을 위한 폴더를 생성하고 해당 폴더들 안에 각 기능을 위한 모듈을 개발한다.

Vue.js에서는 기본적으로 component.ts, service.ts, vue파일이 세트로 묶인다. 여기에 각 페이지별로 뻗어가는 하위 기능을 위한 view는 component와 vue파일만 새로 생성한다.

이제 book-rental이란 모듈을 개발할 예정인데, 이 모듈 내에는 우선적으로 도서 조회, 검색, 도서 세부정보 보기를 만들 것이다. 추후 대여/반납 기능을 추가할 예정이다. 

따라서 아래 이미지와 같이 폴더 및 패키지를 생성한다.


![](/images/judy/2020-07-10-15-06-54.png)