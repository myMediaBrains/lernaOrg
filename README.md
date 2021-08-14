# [Lerna](https://lerna.js.org/)

lerna 는 git 과 npm 으로 멀티-패키지 저장소들을 관리하는데 있어, workflow 를 최적화하는 도구이다.

## Commands

| commnad                      | options                     | 설명                                                                                                                                                     |
| ---------------------------- | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $ leran init                 |                             | 새로운 lerna repo 생성 또는 기존 repo를 현재 lerna 버전으로 upgrade                                                                                      |
|                              | --independent               | independent versioning mode                                                                                                                              |
| $ lerna bootstrap            |                             | current Lerna repo 에서 packages 를 bootstrap 한다.                                                                                                      |
|                              |                             | 모든 packages의 dependencies 를 install 하고, 어떠한 cross-dependencies 도 모두 link 한다.                                                               |
|                              |                             | 이 command 는 매우 중요하다. packages 가 당신의 node_modules 에 있고, 사용가능한 것과 마찬가지로, require() 로 당신의 package names 를 사용할 수 있다.   |
| $ lerna import \<pathToRepo> |                             | 업데이트되어진 packages의 a new release 를 create 한다.                                                                                                  |
|                              |                             | 새로운 버전에 대하여 prompt 하고, git 과 npm 에 모든 packages를 update 한다.                                                                             |
|                              | \--npm-tag [tagname]        | 주어진 npm dist-tag(디폴트는 latest)로 npm 에 publish 한다.                                                                                              |
|                              | \--canary                   | a canary release 를 create 한다.                                                                                                                         |
|                              | \--skip-git                 | 어떠한 git commands 도 run 하지 않는다.                                                                                                                  |
|                              | \--force-publish [packages] | specified packages(comma-seperated) 또는 \* (changed packages 에 대하여 git diff 을 skip 한다.) 을 사용하여 모든 packages 에 대하여 강제로 publish 한다. |
| $ lerna changed              |                             | last release 이후에 change 된 packages 가 무엇인지를 check 한다.                                                                                         |
| $ lerna diff [package?]      |                             | last release 이후에 모든 packages 또는 a single package 를 diff 한다.                                                                                    |
| $ lerna run [script]         |                             | 해당 script 가 있는 각각의 package 에서 an [npm script](https://docs.npmjs.com/misc/scripts) 를 run 한다.                                                |
| $ lerna ls                   |                             | current Lerna repo 에 있는 모든 public packages를 list 한다.                                                                                             |
| $ lerna clean                |                             | root 를 제외한 모든 package 에서 node_modules 를 제거한다.                                                                                               |
| $ lerna publish              |                             | 마지막 릴리즈 이후에 업데이트된 패키지를 배포한다.                                                                                                       |
| $ lerna exec                 |                             | 개별 패키지에서 임의의 커맨드를 실행한다.                                                                                                                |

## monorepo 셋업

| no  | 구분                               | 설명                                              |
| --- | ---------------------------------- | ------------------------------------------------- |
| 1   | $ npm i -g lerna                   | lerna 설치                                        |
| 2   | $ git init lernaOrg && cd lernaOrg | 프로젝트 폴더 생성                                |
| 3   | $ echo “node_modules” > .gitignore |                                                   |
| 4   | $ touch README.md                  |                                                   |
| 5   | $ lerna init                       | a lerna repo로 전환                               |
|     |                                    | packages 폴더, lerna.json, package.json 자동 생성 |
| 6   | $ yarn add lerna -D -W             |                                                   |
| 7   | $ touch lerna.json                 |                                                   |

lerna.json

```json
{
  "packages": ["packages/*"],
  "version": "independent",
  "npmClient": "yarn",
  "useWorkspaces": true
}
```

| no  | 구분                 | 설명 |
| --- | -------------------- | ---- |
| 8   | $ touch package.json |      |

package.json

- private: true 는 root 프로젝트가 NPM으로 publish 되는 것을 방지한다.

```json
{
  "name": "root",
  "private": true,
  "workspaces": ["packages/*"],
  "devDependencies": {
    "lerna": "^4.0.0"
  }
}
```

## 패키지 생성 및 셋업

| no  | 구분                                                    | 설명                                                      |
| --- | ------------------------------------------------------- | --------------------------------------------------------- |
| 1   | $ cd packages                                           |                                                           |
| 2   | $ lerna create log-cli                                  | log-cli 패키지 생성                                       |
| 3   | $ lerna create log-core                                 | log-core 패키지 생성                                      |
| 4   | $ cd lernaOrg                                           | root 폴더                                                 |
| 5   | $ yarn add eslint \--dev \--ignore-workspace-root-check | eslint 를 공통 모듈로 사용                                |
| 6   | $ lerna add commander \--scope=log-cli                  | root 폴더에서 log-cli 패키지에 commander 모듈 설치        |
| 7   | $ lerna add chalk \--scope=log-core                     | root 폴더에서 log-core 패키지에 chalk 모듈 설치           |
| 8   | $ touch log-cli                                         | packages/log-cli/lib/log-cli.js                           |
| 9   | $ touch log-core                                        | packages/log-core/lib/log-core.js                         |
| 10  | $ lerna add log-core \--scope=log-cli                   | **log-cli 에 log-core 를 설치**                           |
|     |                                                         | log-cli의 package.json 의 dependencies 에 log-core 설정됨 |

pacakges/log-core/lib/log-core.js

```tsx
const { red } = require('chalk');

function core() {
  console.log(red('❤  Running Core !!!!!'));
}

module.exports = core;
```

packages/log-cli/lib/log-cli.js

```tsx
#!/usr/bin/env node

const { program } = require('commander');
const LogCore = require('log-core');

// action
program.action((cmd) => LogCore());

program.parse(process.argv);
```

## 실행

| no  | 구분                                     | 설명 |
| --- | ---------------------------------------- | ---- |
| 1   | $ cd lernaOrg                            | root |
| 2   | $ node ./pacakges/log-cli/lib/log-cli.js |      |

```sh
➜  lernaOrg (main) ✗ node packages/log-cli/lib/log-cli.js
❤  Running Core !!!!!
➜  lernaOrg (main) ✗
```

## Git commit과 push

| no  | 구분               | 설명 |
| --- | ------------------ | ---- |
| 1   | $ touch .gitignore | root |

.gitignore

```tsx
.idea
node_modules
*.lock
*.lock.json
```

| no  | 구분                                                              | 설명 |
| --- | ----------------------------------------------------------------- | ---- |
| 2   | $ git add .                                                       |      |
| 3   | $ git commit -m “deploy”                                          |      |
| 4   | $ git remote add origin git@github.com:myMediaBrains/lernaOrg.git |      |
| 5   | $ git push -u origin main                                         |      |

## 패키지 배포

| no  | 구분                        | 설명                                                                                                                                                            |
| --- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | $ lerna publish             | [빠르게 배우는 NPM 패키지 생성부터 배포까지 완벽 가이드](https://kdydesign.github.io/2020/08/28/npm-tutorial/)                                                  |
|     |                             | NPM Repository에 배포하기 전 **log-cli와 log-core의 package.json에서 `name` 속성을 변경**하자. 해당 명칭은 이미 배포된 다른 모듈과 충돌이 나므로 변경해야 한다. |
| 2   | $ npm unpublish log-core -f | NPM Repository 에 배포된 패키지는 72시간이 지나면, 직접 지울 수 없어서 불필요하다면, 즉시 삭제한다.                                                             |
|     | $ npm unpublish log-cli -f  | log-cli 에 log-core가 종속되어 있기 때문에, log-core 를 먼저 삭제해야 한다.                                                                                     |

## 릴리즈 패키지 설치 및 실행

| no  | 구분                                        | 설명                                                                                                                                                 |
| --- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | $ npm install -g log-cli // or PACKAGE_NAME | 실제로 배포했던 패키지를 설치하여 사용하는 경우이다.                                                                                                 |
| 2   | $ touch lerna.json                          | 릴리즈 노트는 보통 CHANGE.md 로 남게 되는데, 이렇게 하기 위해서는 다음과 같이 command의 publish 에 기본적인 [conventionCommit]() 를 추가 설정해준다. |
|     |                                             | 기본적인 옵션외에 [lerna-changelog](https://github.com/lerna/lerna-changelog)를 사용하면, 더 적합한 릴리즈 노트를 작성할 수 있다.                    |

lerna.json

```json
{
  "version": "independent",
  "npmClient": "yarn",
  "useWorkspaces": true,
  "packages": ["packages/*"],
  "command": {
    "publish": {
      "conventionalCommits": true
    }
  }
}
```
