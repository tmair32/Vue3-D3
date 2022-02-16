# Vue3 + D3.js

>   Vue 3,TypeScript, D3.js를 가지고 놀아보자
>
>   css는 windicss 사용 예정

*   [1. Vue 3 설치](#Vue 3 설치)

    

*   [2. Vue UI](#Vue UI)

    

*   [3. 각종 설정](#각종 설정)

    

*   [4. D3 차트 그리기]()



## Vue 3 설치

```
$ npm install -g @vue/cli
$ vue create project-name
```

project-name으로 vue 프로젝트 생성

```
Vue CLI v4.5.15
? Please pick a preset: (Use arrow keys)
> vue 3 custom ([Vue 3] node-sass, babel, typescript, router, vuex, eslint)
  Default ([Vue 2] babel, eslint)
  Default (Vue 3) ([Vue 3] babel, eslint)
  Manually select features
```

Manually select features를 골라 내 입맛대로 프로젝트 세팅

```
 (*) Choose Vue version
 (*) Babel
 (*) TypeScript
 ( ) Progressive Web App (PWA) Support
>( ) Router
 ( ) Vuex
 ( ) CSS Pre-processors
 (*) Linter / Formatter
 ( ) Unit Testing
 ( ) E2E Testing
```

vue router와 vuex, css pre-processors도 보통 설치하지만 vue cli로 설치하면 에러가 날때도 있고.. 이번 프로젝트에선 전부 필요 없는 기능들이기 때문에 생략한다.

```
? Check the features needed for your project: Choose Vue version, Babel, TS, Linter
? Choose a version of Vue.js that you want to start the project with (Use arrow keys)
> 2.x 
  3.x 
```

Vue 3를 사용할 것이기 때문에 3.x 선택

```
? Use class-style component syntax? (y/N) 
```

class-style component syntax라고 vue를 사용하는 방법의 일종인데 할 줄 모르니까 N을 선택한다.

```
? Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)? (Y/n) 
```

무언가 어려워 보이는데 TypeScript에도 Babel을 사용할 것인지 묻는 것이다. Babel 세팅에 자신 없으면 Y를 골라주자.

```
? Pick a linter / formatter config: 
  ESLint with error prevention only 
  ESLint + Airbnb config 
  ESLint + Standard config 
  ESLint + Prettier 
> TSLint (deprecated) 
```

eslint 설정에 관해 고르는 것이다. 난 ESLint + Prettier를 즐겨쓴다.

```
? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)
>(*) Lint on save
 ( ) Lint and fix on commit
```

Lint를 언제 돌릴 것인지이다. 둘 다 선택도 가능하고 하나씩도 가능하다.

```
? Where do you prefer placing config for Babel, ESLint, etc.? (Use arrow keys)
> In dedicated config files 
  In package.json 
```

Babel, ESLint 등등의 setting을 개별적으로 하는게 좋은지 package.json에서 일괄적으로 하는게 좋은지 고르는 것이다. 따로 하는걸 추천한다.



이 뒤에 이 세팅을 저장할까 묻는데 알아서 하도록 하자.



설치가 끝난 후에는

```
npm run serve
```

를 쳐서 서버에 잘 접속이 되는지 확인해보자.

![vue-welcome](https://kangwoojin.github.io/assets/images/vuejs/vue-welcome.png)



이 화면이 나오면 성공한 것이다.



## Vue UI

```bash
$ vue ui
```

![image-20220216142426613](C:/Users/tmair/AppData/Roaming/Typora/typora-user-images/image-20220216142426613.png)

<img src="C:/Users/tmair/AppData/Roaming/Typora/typora-user-images/image-20220216142504416.png" alt="image-20220216142504416" style="zoom:150%;" />



플러그인 > `+ 플러그인 추가`

`@vue/cli-plugin-router`를 설치한다.

`vue-cli-plugin-windicss`를 추가로 설치한다.

>   vuex, router 등등 기본적인 플러그인이 추가로 필요할 경우 vue ui를 통해 설치하는게 깔끔한 것 같다.



의존성 > `+ 의존성 설치` > 개발 의존성

`d3`를 설치한다.



## 각종 설정

### Vue 3

```js
// vue.config.js
module.exports = {
  outputDir: "./dist/",
  pages: {
    index: "src/main.ts",
  },
};
```

vue를 사용하면서 웹팩 설정을 하는 곳이다.

기본적인 웹팩을 몰라도 vue가 어지간한건 웹팩에 맞게끔 설정할 수 있게 도와준다.

이번 프로젝트에서는 별로 설정할게 없다.



### ESLint-Prettier

```javascript
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    node: true,
  },
  extends: [
    "plugin:vue/vue3-essential",
    "eslint:recommended",
    "@vue/typescript/recommended",
    "@vue/prettier",
    "@vue/prettier/@typescript-eslint",
  ],
  parserOptions: {
    ecmaVersion: 2020,
  },
  rules: {
    "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
    "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off",
  },
};

```

vue cli를 통해 lint 설치시 기본적으로 세팅되어 있는 모습이다.

나는 여기에 추가적인 작업 몇가지만 할 예정이다.

```javascript
module.exports = {
    ...
    rules: {
        "prettier/prettier": ["error", { endOfLine: "auto" }],
    }
    ...
}
```

마지막 줄에 억까 에러가 잡히는 경우가 있는데 이 코드를 달면 기본적으로 제거가 되고, 그래도 남는 경우에는 VSCode 혹은 IDE 우측 하단에 Select End of Line Sequence를 LF -> CRLF로 바꾸면 된다.

```json
// settings.json
{
    "vue/require-default-prop": "off",
    "editor.formatOnSave": false,
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    "[vue]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[javascript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
}
```

이와 같은 내용을 추가한다.

ESLint와 Prettier 사이에 충돌을 잡아준다.

```json
// package.json
{
    ...
    "scripts": {
        "lint": "eslint --ext .js,.vue --ignore-path .gitignore --fix src",
        "format": "prettier . --write"
    }
    ...
}
```

마지막으로 package.json에 lint와 format 코드를 추가해주고 lint가 이상할 때 마다 한번씩 `npm run lint`를 돌려주자.

>   ESLint는 팀 혹은 사용자마다 세팅이 각양각생인데 이를 Rules 혹은 Plugin을 잘 통일해서 쓰면 협업이 무척이나 쉬워진다. 나는 그냥 최대한 간단하게 해놓고 기본적인 기능 위주로 쓴다.



### windicss 설정

>   windicss를 고른 이유는 tailwind css는 사용 해봤고 tailwind와의 차별점은 class명에 동적 값을 집어 넣을 수 있는 것이다.
>
>   물론 이는 vue 3로 넘어오면서 처리할 수 있게 되었다지만, 새로우니까 써본다.
>
>   그리고 잘 쓰면 tailwind에서 일어난 무한 class(@apply를 통해 해결은 가능하다.)를 좀 줄일 수 있을 것 같다.
>
>   솔직히 그냥 써보고 싶어서 써보는거다.
>
>   물론 Vue 3의 styled scoped 기능 혹은 react의 경우 styled components가 있지만, 이는 개발자 친화적인 프레임워크일 수는 있지만 퍼블리셔와 연계할 경우 좋다고 생각하지 않는다.

```typescript
//windi.config.ts(js)
import { defineConfig } from "windicss/helpers";
import colors from "windicss/colors";
import plugin from "windicss/plugin";

export default defineConfig({
  darkMode: "class", // or 'media'
  theme: {
    extend: {
      screens: {
        sm: "640px",
        md: "768px",
        lg: "1024px",
        xl: "1280px",
        "2xl": "1536px",
      },
      colors: {
        gray: colors.coolGray,
        blue: colors.sky,
        red: colors.rose,
        pink: colors.fuchsia,
        violet: colors.violet,
      },
      fontFamily: {
        sans: ["Graphik", "sans-serif"],
        serif: ["Merriweather", "serif"],
      },
      spacing: {
        128: "32rem",
        144: "36rem",
      },
      borderRadius: {
        "4xl": "2rem",
      },
    },
  },
  plugins: [
    plugin(({ addUtilities }) => {
      const newUtilities = {
        ".skew-10deg": {
          transform: "skewY(-10deg)",
        },
        ".skew-15deg": {
          transform: "skewY(-15deg)",
        },
      };
      addUtilities(newUtilities);
    }),
    plugin(({ addComponents }) => {
      const buttons = {
        ".btn": {
          padding: ".5rem 1rem",
          borderRadius: ".25rem",
          fontWeight: "600",
        },
        ".btn-blue": {
          backgroundColor: "#3490dc",
          color: "#fff",
          "&:hover": {
            backgroundColor: "#2779bd",
          },
        },
        ".btn-red": {
          backgroundColor: "#e3342f",
          color: "#fff",
          "&:hover": {
            backgroundColor: "#cc1f1a",
          },
        },
      };
      addComponents(buttons);
    }),
    plugin(({ addDynamic, variants }) => {
      addDynamic(
        "skew",
        ({ Utility, Style }) => {
          return Utility.handler
            .handleStatic(Style("skew"))
            .handleNumber(0, 360, "int", (number) => `skewY(-${number}deg)`)
            .createProperty("transform");
        },
        variants("skew")
      );
    }),
    require("windicss/plugin/filters"),
    require("windicss/plugin/forms"),
    require("windicss/plugin/aspect-ratio"),
    require("windicss/plugin/line-clamp"),
    require("windicss/plugin/typography")({
      modifiers: ["DEFAULT", "sm", "lg", "red"],
    }),
  ],
});
```

tailwind처럼 config 파일을 통해 플러그인 확장이 가능하다.



### D3 차트 그리기

