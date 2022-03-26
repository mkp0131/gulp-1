# gulp

## Setting

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
