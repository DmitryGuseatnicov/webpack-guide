# webpack-guide
Гайд по сборке многостраничного сайта + линтеры (pug, scss, babel, eslint, stylelint, prittier)

## Устанив все необходимые пакеты:

  #### webpack:
  ```
  npm i --save -D webpack webpack-cli webpack-dev-server cross-env
  ```
    
  #### pug:
  ```
  npm i --save -D pug simple-pug-loader simple-pug-loader html-webpack-plugin
  ```
  
  #### scss:
  ```
  npm i --save -D sass sass-loader postcss postcss-loader postcss-preset-env css-loader autoprefixerresolve-url-loader mini-css-extract-plugin
  ```
    
  #### babel:
  ```
  npm i --save -D @babel/core @babel/preset-env babel-loader babel-eslint
  ```
    
  #### eslint:
  ```
  npm i --save -D eslint@7.0.0 eslint-config-airbnb eslint-config-airbnb-base eslint-plugin-importeslint-plugin-fsd --force
  ```
    
  #### stylelint:
  ```
  npm i --save -D stylelint stylelint-config-airbnb stylelint-config-rational-order stylelint-declaration-block-no-ignored-properties stylelint-order stylelint-scss
  ```
  
  #### prettier:
  ```
  npm i --save -D prettier eslint-config-prettier eslint-plugin-prettier prettier-eslint stylelint-config-prettier stylelint-prettier --force
  ```
    
## Структура файлов
```
  src / -- корневая папка 
    pages/ 
      page-name/ -- папки с файлами для каждой отдельной страницы(page-name меняем на наше нужное название)
        page-name.pug
        page-name.scss
        page-name.js
    styles/
      -- файлы со шрифтами, переменными и минимальными  глобальными стилями
    components/
      component-name/ -- папки компонентов (component-name меняем на наше нужно название)
  
```   
## Кофиги:
  в корне проекта создаем файлы .eslintignore, .eslintrc, .gitignore, .prettierrc, .stylelintrc, webpack.config.js
   #### .gitignore:
   ```
    node_modules
    dist
   ```
 
   #### .eslintignore:
   ```
    webpack.config.js
   ```
  
   #### webpack.config.js :
  ```
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const MiniCssExtractPlugin = require('mini-css-extract-plugin');
  const fs = require('fs');

  let mode = 'development';
  if (process.env.NODE_ENV == 'production') {
  mode = 'production';
  }
  console.log(mode + ' mode');

  // тут получаем пути до js файлов и создаем entryPoints для каждой страницы
  const PAGES_DIR = path.resolve(__dirname, 'src/pages');
  const PAGES = fs.readdirSync(path.resolve(__dirname, 'src/pages'));
  const entryPoints = Object.assign(
  {},
  ...PAGES.map((page) => ({
    [page]: `${PAGES_DIR}/${page}/${page}.js`,
  }))
  );

  module.exports = {
    mode: mode,
    context: path.resolve(__dirname, 'src'),
    entry: entryPoints,
    output: {
      filename: '[name].[contenthash].js',
      path: path.resolve(__dirname, 'dist'),
      clean: true,
    },
    devServer: {
      historyApiFallback: true,
      static: {
      directory: path.resolve(__dirname, 'dist'),
      },
      open: '/index.html',
      hot: false,
    },
    devtool: 'source-map',
    plugins: [
      new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
      }),
    // тут каждую страницу передаем в HtmlWebpackPlugin
    ...PAGES.map(
      (page) =>
        new HtmlWebpackPlugin({
          filename: `${page}.html`,
          template: `${PAGES_DIR}/${page}/${page}.pug`,
          chunks: [page],
          inject: 'body',
        })
    ),
    ],
    module: {
    rules: [
      {
        test: /\.(sa|sc|c)ss$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  [
                    'autoprefixer',
                    {
                      overrideBrowserslist: ['> 1%', 'last 2 versions'],
                    },
                    'postcss-preset-env',
                  ],
                ],
              },
            },
          },
          'resolve-url-loader',
          'sass-loader',
        ],
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'assets/images/[name][ext]',
        },
      },
      {
        test: /\.ico/i,
        type: 'asset/resource',
        generator: {
          filename: 'assets/favicon/[name][ext]',
        },
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'assets/fonts/[name][ext]',
        },
      },
      {
        test: /\.pug$/,
        use: [
          {
            loader: 'simple-pug-loader',
            options: {
              pretty: true,
            },
          },
        ],
      },
      {
        test: /\.m?js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
          },
        },
      },
    ],
    },
  };
  ```
 #### .eslintrc:
 ```
  {
  "parserOptions": {
    "parser": "babel-eslint",
    "sourceType": "module",
    "allowImportExportEverywhere": true,
    "ecmaVersion": 2015
  },
  "env": {
    "browser": true,
    "es6": true,
    "jquery": true,
    "node": true
  },
  "extends": [
    "airbnb-base",
    "plugin:fsd/all",
    "plugin:prettier/recommended",
    "prettier"
  ],
  "globals": {
    "$": "readonly",
    "jQuery": "readonly"
  },
  "plugins": ["fsd", "pug", "prettier"],
  "rules": {
    "no-new": "off",
    "import/no-unresolved": "off",
    "prefer-regex-literals": "off",
    "no-underscore-dangle": ["off", { "allow": true }],
    "prettier/prettier": "error"
  }
}
 ```
 
 #### .stylelintrc:
 ```
  {
    "extends":[
      "stylelint-config-airbnb", 
      "stylelint-config-rational-order",
      "stylelint-prettier/recommended"
    ],
    "plugins": [
      "stylelint-scss",
      "stylelint-order",
      "stylelint-declaration-block-no-ignored-properties",
      "stylelint-config-rational-order/plugin",
      "stylelint-prettier"
    ],
    "rules": {
      "prettier/prettier": [true],
      "order/properties-order": [],
      "plugin/rational-order": [true, {
          "border-in-box-model": false,
          "empty-line-between-groups": false
      }],
      "max-nesting-depth": 3,
      "declaration-property-value-blacklist": {},
      "rule-empty-line-before": [
        "always-multi-line",
        {
          "except": [
            "first-nested"
          ],
          "ignore": [
            "after-comment"
          ]
        }
      ],
      "order/order": [
        "custom-properties",
        "dollar-variables",
        "declarations",
        "rules",
        "at-rules",
        {
          "type": "at-rule",
          "name": "media"
        }
      ]
    }
  }
 ```
 #### .prettierrc.json:
 ```
  {
    "singleQuote": true,
    "arrowParens": "always",
    "max-len": ["error", 80, 2],
    "printWidth": 80,
    "tabWidth": 2,
    "useTabs": false,
    "endOfLine": "auto",
    "overrides": [
      {
        "files": "*.pug",
        "options": {
          "parser": "pug",
          "singleQuote": true,
          "max-len": ["error", 80, 2],
          "printWidth": 80
        }
      }
    ]
  }
 ```
 
 ##Scripts: 
  ну и добависм скрипты для запуска webpack и linter-ов в package.json
  
  ```
  "scripts": {
    "start": "cross-env NODE_ENV=development webpack serve",
    "dev": "cross-env NODE_ENV=development webpack",
    "build": "cross-env NODE_ENV=production webpack",
    "deploy": "gh-pages -d dist",
    "lint": "eslint src",
    "lint:fix": "eslint src --fix",
    "stylelint": "stylelint \"src/**/*.scss\"",
    "stylelint:fix": "stylelint \"src/**/*.scss\" --fix"
  }
  ```
