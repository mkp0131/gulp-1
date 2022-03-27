# gulp

## 세팅

- gulp 설치

```js
npm install --global gulp-cli // 전역설치
gulp --version // 버전확인
```

- babel 설치 / .babelrc

- gulpfile.babel.js 파일생성

- package.json 에 ["dev": "gulp dev"] 추가

- gulpfile.babel.js 파일에 dev 함수를 생성후 export, "npm run dev" 실행시 dev 함수가 실행됨.

```js
  "scripts": {
    "dev": "gulp dev",
    "build": "gulp build"
  },
```

```js
export const dev = () => console.log('dev');
```

> export 된 함수들은 package.json 에서 사용가능

## 사용법

### gulp.series

- dev 값을 gulp.series([실행할 함수리스트]) 의 반환 값으로 부여

```js
// gulpfile.babel.js 파일내 clean, pug 함수가 순서대로 실행
export const dev = gulp.series([clean, pug]);
```

- gulp.series 는 파이프 함수를 반환
- 파라미터로 파이프함수도 받을 수 있다.

```js
const prepare = gulp.series([clean]);

const assets = gulp.series([pug]);

export const dev = gulp.series([prepare, assets]);
```

### gulp.src(파일경로).pipe(함수실행)

- gulp.src(파일경로).pipe(함수실행) 를 이용하여 최종 배포파일을 생성

```js
// 기본경로를 routes 로 묶어준다.
const routes = {
  pug: {
    src: 'src/*.pug',
    dest: 'build',
  },
};

const pug = () =>
  gulp.src(routes.pug.src).pipe(gpug()).pipe(gulp.dest(routes.pug.dest));
```

### 파일경로 설정

- 파일 경로 설정시 폴더 안에 모든 걸 탐색(depth 에 상과없이 모두)하고 싶을때는 '/\*\*/' 를 사용한다.
- '/\*/' 의 경우 그 어떤 폴더라도 한개의 depth 만 탐색

```js
watch: 'src/**/*.pug', // src/partials/ttt/test.pug 까지 depth 에 상관없이 실행.
```

### gulp.task('이름', 함수)

- task 의 첫번재 인자로 변수를 등록, gulp.series([task 이름]) 의 인자로 사용 가능하다
- const 로 등록하면 되니 필요없는듯?

```js
// 결과물을 바로 보고 싶을 경우 gulp-webserver 을 설치
// 아래의 두개 코드가 동일하게 동작한다.
gulp.task('webserver', function () {
  gulp.src('build').pipe(
    webserver({
      livereload: true,
      // open: true,
    })
  );
});

const webserver = () => {};
```

### gulp.parallel([함수리스트])

- gulp.series 와 동일하지만 실행이 거의 동시? 에 된다.

### gulp.watch(경로, 콜백함수)

- 경로에 주어진 파일이 수정되면, 콜백함수가 실행된다.

```js
const watch = () => {
  gulp.watch(routes.pug.watch, pug);
};
```

### gulp-image [npm package]

- 이미지를 모아주는 패키지
- 이미지는 용량이 클 경우 watch 할때 느려지니 주위가 필요

```js
import image from 'gulp-image';

const img = () =>
  gulp.src(routes.img.src).pipe(image()).pipe(gulp.dest(routes.img.dest));
```

### gulp-sass

- scss 를 css 로변환
- sass().on('error', sass.logError) 는 작동이 멈추는 error 가 아닌 css 속성에 관한 경고

```js
import dartSass from 'sass';
import gulpSass from 'gulp-sass';
const sass = gulpSass(dartSass);

const scss = () =>
  gulp
    .src(routes.scss.src)
    .pipe(sass().on('error', sass.logError))
    .pipe(gulp.dest(routes.scss.dest));
```

- scss 에서 scss 를 import 해서 사용할때 파일 이름 앞에 '\_' 를 붙여주면 각각의 파일을 생성 하는 것이 아니라 하나의 파일로 생성

```
_reset.scss
style.scss (@import '_reset';)
=>
style.css 로 하나로 통합해서 생성
```

### gulp-autoprefixer [npm package]

- css 의 호환성 생성(IE, SAFARI 등)
- package.json 에서 "browserslist" 속성 추가

```json
"browserslist": [
  "last 2 versions"
],
```

- scss 파이프에 autoprefixer() 추가

```js
const scss = () =>
  gulp
    .src(routes.scss.src)
    .pipe(sass().on('error', sass.logError)) // 추가코드
    .pipe(autoprefixer())
    .pipe(gulp.dest(routes.scss.dest));
```

### gulp-csso [npm package]

- css 를 minified 실행
- sourceMap 도 생성 가능
- scss 파이프에 csso() 추가

```js
const scss = () =>
  gulp
    .src(routes.scss.src)
    .pipe(sass().on('error', sass.logError))
    .pipe(autoprefixer())
    .pipe(csso()) // 추가코드
    .pipe(gulp.dest(routes.scss.dest));
```

### gulp-browserify

- Browserify는 개발자가 브라우저에서 사용하기 위해 컴파일하는 Node.js 스타일 모듈을 작성하고 사용할 수 있게 해주는 오픈 소스 JavaScript 번들러
- gulp 용 browserify 설치
- NODEJS 로 컴파일된 것을 babel 로 다시 컴파일
- 바벨로 컴파일 해주는 'babelify', 코드를 압축시켜주는 'uglifyify' 설치

```js
const script = () =>
  gulp
    .src(routes.js.src)
    .pipe(
      bro({
        transform: [
          babelify.configure({ presets: ['@babel/preset-env'] }),
          ['uglifyify', { global: true }],
        ],
      })
    )
    .pipe(gulp.dest(routes.js.dest));
```

### routes 경로 설정 팁

#### src 경로 설정시

- 파일 내에서 import OR include 해서 사용하는 파일, 그리고 파일하나만 export 하고 싶은 파일류는 index(main) 파일 하나만 src 로 넣어준다.

#### watch 경로 설정시

- scss, js, pug 같은 경우 depth 가 생길 가능성이 있는 파일들은 '/\*\*/' 을 사용하여 처리해준다.
- 또한 index 파일 하나만 watch 하는 것이 아닌 하위폴더 모든 것을 다 설정한다.

```js
const routes = {
  pug: {
    watch: 'src/**/*.pug',
    src: 'src/*.pug',
    dest: 'build',
  },
  img: {
    src: 'src/img/*',
    dest: 'build/img',
  },
  scss: {
    watch: 'src/scss/**/*.scss',
    src: 'src/scss/style.scss',
    dest: 'build/css',
  },
  js: {
    watch: 'src/js/**/*.js',
    src: 'src/js/main.js',
    dest: 'build/js',
  },
};
```

### package.json - script 생성

- gulpfile.babel.js 파일에서 export 한 값들을 package.json 에서 명령어로 사용

```js
// gulpfile.babel.js
export const build = gulp.series([prepare, assets]);
export const dev = gulp.series([build, live]);
export const deploy = gulp.series([build, gh]);
```

```json
// package.json
"scripts": {
  "dev": "gulp dev",
  "build": "gulp build",
  "deploy": "gulp deploy"
},
```
