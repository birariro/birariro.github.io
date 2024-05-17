# Custom GitHub Action 으로 Chat-GPT 코드리뷰 제작하기 


![](2e98c2e1-49ca-4c59-bfc9-ceb7d7008381.png)

github에 pull request를 날리면 chat gpt 가 코드리뷰를 해주는 

custom github action 을 만들려 한다.

만드는 방법은 아래의 링크를 참고할 수 있다.

[https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)

 [Creating a JavaScript action - GitHub Docs

In this guide, you'll learn about the basic components needed to create and use a packaged JavaScript action. To focus this guide on the components needed to package the action, the functionality of the action's code is minimal. The action prints "Hello Wo

docs.github.com](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)

먼저 가볍게 github action을 만드는 방법을 알아보자.

### action.yml

github Action의 이름, 설명, input, output, icon 등 다양한 중요정보를 기입하는 공간이다.

    name: "chicken-code-review"
    description: "꿩 대신 닭이라는 심정으로 Chat-GPT 에게 코드 리뷰를 요청한다."
    
    inputs:
      github-token:
        description: GitHub WebHook Secret Token
        required: true
    
    
    runs:
      using: "node16"
      main: "src/index.js"
    
    branding: 
      icon: 'terminal'
      color: 'orange'

action의 이름과 설명이 들어있는 것을 볼 수 있고

input으로 github token을 받으며 이 파라미터는 필수(required)라고 명시하였다.

### index.js

@actions/core 는 input, output 등을 편하게 관리할 수 있도록 도와주는 라이브러리이다.

@actgion/github 는 Octokit 인스턴스를 사용하여 github를 제어할 수 있게 도와주는 라이브러리이다.

precess.env 에서 제공하는 환경변수로 다양한 정보를 얻게 되는데 얻을 수 있는 정보는 아래에서 확인 가능하다

[https://docs.github.com/en/actions/learn-github-actions/variables](https://docs.github.com/en/actions/learn-github-actions/variables)

 [Variables - GitHub Docs

Variables provide a way to store and reuse non-sensitive configuration information. You can store any configuration data such as compiler flags, usernames, or server names as variables. Variables are interpolated on the runner machine that runs your workfl

docs.github.com](https://docs.github.com/en/actions/learn-github-actions/variables)

    const core = require("@actions/core");
    const github = require("@actions/github");
    
    async function run() {
    
      try {
        const token = core.getInput("github-token");
     
        const { GITHUB_REPOSITORY, GITHUB_SHA } = process.env;
        const [owner, repo] = GITHUB_REPOSITORY.split("/");
        const sha = GITHUB_SHA;
        const client = new github.GitHub(token);
        
        await client.request(`POST /repos/${owner}/${repo}/commits/${sha}/comments`, {
            body: "hello",
        });
    
      } catch (error) {
        core.setFailed(error.message);
      }
    }
    
    run();

이제 다른 프로젝트에서 해당 action을 사용해 보면

    name: PUSH EVENT
    on: [push]
    
    jobs:
      reviewer:
        runs-on: ubuntu-latest
        steps:
          - name: reviewer run
            uses: birariro/chicken-reviewer@1.0.0
            with:
              github-token: ${{ secrets.GIT_TOKEN }}

![](7e9b5b50-db7c-45ec-8b3d-ead5410674c3.png)

hello !

자 그럼 이제 처음 목적이었던 chat-GPT에게 코드 리뷰를 받기 위해서는

1\. commit에서 변경사항 정보를 얻는다.

2\. chat-GPT에게 물어본다.

3\. 결과를 commit 댓글에 달아준다.

가 끝이다.

차근차근 해보자.

commit의 변경 사항을 가지고 온 후에

    const compare = await client.request(`GET /repos/${owner}/${repo}/compare/${sha}^...${sha}`);
    const changes = compare.data.files.map((file) => file.filename);
    
    let fileDiff = ""
    for (const file of changes) {
      const diff = await client.request(`GET /repos/${owner}/${repo}/commits/${sha}`, {
        headers: {
          Accept: "application/vnd.github.diff",
        },
      });
    
      fileDiff += `${diff.data}\n\n`
    }

적절한 prompt를 넣어서 변경사항에 대한 코드리뷰를 받고

    let _prompt = prompt(language) + fileDiff;
    
    const conf = new Configuration({
        apiKey:key
    });
    
    const openai = new OpenAIApi(conf);
    const chatCompletion = await openai.createChatCompletion({
        model: "gpt-3.5-turbo",
        messages: [{role: "system", content: _prompt}],
    });
    
    return chatCompletion.data.choices[0].message.content

pull request에 댓글을 달아주면 된다.

    const pullRequests = await client.pulls.list({
        owner: owner,
        repo: repo,
        head: sha,
    });
    if (pullRequests.data.length > 0) {
        const pullRequestNumber = pullRequests.data[0].number;
        await client.request(`POST /repos/${owner}/${repo}/issues/${pullRequestNumber}/comments`, {
            body: review,
        });
        core.info('Comment added to the pull request.');
    }

![](f9318d87-3beb-47e4-8d87-2bec9d2c0a53.png)

전체코드는 아래에서 볼 수 있다.

[https://github.com/birariro/chicken-code-review](https://github.com/birariro/chicken-code-review)

 [GitHub - birariro/chicken-code-review: Get code reviews from chat-GPT

Get code reviews from chat-GPT. Contribute to birariro/chicken-code-review development by creating an account on GitHub.

github.com](https://github.com/birariro/chicken-code-review)

### node\_modules 가 아닌 ncc로 업로드

[https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action#commit-and-push-your-action-to-github](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action#commit-and-push-your-action-to-github)

 [Creating a JavaScript action - GitHub Docs

In this guide, you'll learn about the basic components needed to create and use a packaged JavaScript action. To focus this guide on the components needed to package the action, the functionality of the action's code is minimal. The action prints "Hello Wo

docs.github.com](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action#commit-and-push-your-action-to-github)

위의 글을 보면 node\_modules를 업로드하거나

ncc를 사용하여 dist를 업로드하는 방법 둘 중 하나를 사용하라고 한다.

ncc를 사용하려면

package.json 에서 ncc로 build 하는 scripts를 추가한 후에

    npm i -g @vercel/ncc

    {
      "name": "chicken-code-review",
      "version": "1.0.0",
      "description": "If you do a PR, you'll get a code review",
      "main": "src/index.js",
      "scripts": {
        "test": "jest",
        "build": "ncc build src/index.js --license licenses.txt"
      },

action.yml 에서 main을 빌드된 dist의 index로 변경한다

    runs:
      using: "node16"
      main: "dist/index.js"

node\_modules는 이제 github에 업로드하지 않고

dist 파일을 업로드한다.