# Git Hook 을 멋들어지게 즐겨보기 


Hook ?!
-------

Git 에서 어떠한 이벤트가 생겼을때 자동으로 특정 스크립트이다.

Git 으로 형상관리 하는 프로젝트에는 .git 디렉토리가 있으며 그 밑에는 hooks 디렉토리가 있으며

그 안에 유용한 샘플 hook들이 들어있다.

hook은 이벤트 별로 다양한 이름을 가지며 본 글에서 사용할 hook으로는

커밋 메시지 작성전에 동작하는 pre-commit,

커밋 메시지 편집기가 실행하기 직전에 동작하는 prepare-commit-msg

커밋 메시지 작성후 커밋이 완료되기 직전인 commit-msg

이렇게 3개를 활용 해보려한다.

### commit-msg 로 커밋 메시지 검사

[https://www.conventionalcommits.org/en/v1.0.0/](https://www.conventionalcommits.org/en/v1.0.0/)

 [Conventional Commits

A specification for adding human and machine readable meaning to commit messages

www.conventionalcommits.org](https://www.conventionalcommits.org/en/v1.0.0/)

위의 글을 참고해서 git commit 컨벤션을 사용하려하는데

hook 를 통해 해당 컨벤션을 지키기 않으면 커밋을 할 수 없게 하려한다.

    <type>[optional scope]: <description>
    
    [optional body]
    
    [optional footer(s)]

위와같은 형태를 가진 commit 메시지로 작성되었는지 보기위해

optional 인 body 와 footer 은 하지않고

type 에 정해진 키워드가 들어갔는지 검사하려한다.

type 에 사용가능한 키워드는 

feat, fix,docs, style, refactor, test, chore 로 제한한다.

또한 키워드에 맞게 입력하면 이모지를 주입해서 보기좋은 커밋메시지를 완성시켜 준다.

    #!/bin/sh
    
    COMMIT_MSG_FILE=$1
    
    #커밋 메시지중 1번 라인을 얻는다.
    FIRST_LINE=`head -n1 ${COMMIT_MSG_FILE}`
    
    byPass() {
    
      if [[ $FIRST_LINE =~ ^(Merge branch) ]] ||
         [[ $FIRST_LINE =~ ^(Merge pull request) ]]; then
    
        echo "[commit lint pass] auto generated commit message"
        exit 0
      fi
    
      if [[ $FIRST_LINE =~ ^(initial) ]]; then
        echo "[commit lint pass] init commit message"
        exit 0
      fi
    }
    
    validHeader() {
      if [[ $FIRST_LINE == "" ]]; then
        echo "[commit lint error] empty commit message"
        exit 1
      fi
    
    
      if [[ $FIRST_LINE =~ (\.)$ ]]; then
        echo "[commit lint error] 문장 마지막의 ('.') 마침표를 제거"
        exit 1
    
      elif [[ ! $FIRST_LINE =~ ^(feat(\(.*\))?!?: ) ]] &&
          [[ ! $FIRST_LINE =~ ^(fix(\(.*\))?!?: ) ]] &&
          [[ ! $FIRST_LINE =~ ^(docs(\(.*\))?!?: ) ]] &&
          [[ ! $FIRST_LINE =~ ^(style(\(.*\))?!?: ) ]] &&
          [[ ! $FIRST_LINE =~ ^(refactor(\(.*\))?!?: ) ]] &&
          [[ ! $FIRST_LINE =~ ^(test(\(.*\))?!?: ) ]] &&
          [[ ! $FIRST_LINE =~ ^(chore(\(.*\))?!?: ) ]]; then
    
        echo "\n[commit lint error] HEADER 의 type, 콜론, 띄어쓰기 형태를 확인\n"
    
        echo "<type>(scope option): <subject>"
        echo "- feat: 새로운 기능 추가"
        echo "- fix: 버그 수정"
        echo "- docs: 문서의 수정"
        echo "- style: (코드의 수정 없이) 스타일(style)만 변경(들여쓰기 같은 포맷이나 세미콜론을 빼먹은 경우)"
        echo "- refactor: 코드를 리펙토링"
        echo "- test: Test 관련한 코드의 추가, 수정"
        echo "- chore: (코드의 수정 없이) 설정을 변경"
        exit 1
    
      fi
    }
    
    injectHeaderEmoji() {
      COMMIT_MESSAGE=`head ${COMMIT_MSG_FILE}`
      if [[ $FIRST_LINE =~ ^(feat.*) ]]; then
        NEW_COMMIT_MESSAGE="🚀 $COMMIT_MESSAGE"
      elif [[ $FIRST_LINE =~ ^(fix.*) ]]; then
        NEW_COMMIT_MESSAGE="🔥 $COMMIT_MESSAGE"
      elif [[ $FIRST_LINE =~ ^(docs.*) ]]; then
        NEW_COMMIT_MESSAGE="📝 $COMMIT_MESSAGE"
      elif [[ $FIRST_LINE =~ ^(style.*) ]]; then
        NEW_COMMIT_MESSAGE="🎨 $COMMIT_MESSAGE"
      elif [[ $FIRST_LINE =~ ^(refactor.*) ]]; then
        NEW_COMMIT_MESSAGE="🧠 $COMMIT_MESSAGE"
      elif [[ $FIRST_LINE =~ ^(test.*) ]]; then
        NEW_COMMIT_MESSAGE="🧪 $COMMIT_MESSAGE"
      elif [[ $FIRST_LINE =~ ^(chore.*) ]]; then
        NEW_COMMIT_MESSAGE="🍎 $COMMIT_MESSAGE"
      fi
    
      echo "$NEW_COMMIT_MESSAGE" > $COMMIT_MSG_FILE
    }
    
    byPass
    validHeader
    injectHeaderEmoji
    
    echo "[commit lint pass] lint success"
    exit 0

이제 잘못된 commit message 를 작성하면

![](5448357e-f77c-4309-9635-757ebd33d9fd.png)

캬

commit 작업을 중지시키면서 어떻게 메시지를 작성해야하는지 알려준다.

정상적인 commit message 를 작성하면

![](f2dce64a-a08d-4afd-b57a-07ce1a607994.png)

![](d9b06168-78cd-4c5b-8d7d-5ec2cda5eaff.png)

정상적으로 커밋이 되면서 앞에 이모지를 붙혀준다.

### prepare-commit-msg 로 커밋메시지에 footer 자동 추가

feature 브랜치 네이밍을  **feature/<ticket-number>/<branch-name>** 으로 네이밍하는경우

중간에 있는 이름을 파싱해서

commit 할때 commit message 밑에 자동으로

    footer: ticket-number

위와같이 붙혀주려한다.

    #!/bin/sh
    
    COMMIT_MESSAGE=$1
    
    addCommitFooterTicket() {
      NAME=$(git branch | grep '*' | sed 's/* //')
    
      ISSUE_NUMBER=`echo $NAME | cut -d '/' -f2`
      DESCRIPTION=$(git config branch."$NAME".description)
    
      if [[ $ISSUE_NUMBER == *ISSUE* ]] || [[ $ISSUE_NUMBER == *TIKET* ]]; then
        echo "ticket: $ISSUE_NUMBER $(cat $COMMIT_MESSAGE)" > $COMMIT_MESSAGE
      fi
    
      if [ -n "$DESCRIPTION" ]
      then
         echo "" >> $COMMIT_MESSAGE
         echo $DESCRIPTION >> $COMMIT_MESSAGE
      fi
    }
    
    MERGE=$(cat $COMMIT_MESSAGE|grep -i 'merge'|wc -l)
    
    if [ $MERGE -eq 0 ] ; then
      addCommitFooterTicket
    fi

해당 브랜치명에서 '/' 를 기준으로 두번째 문자열을 얻고

그 두번째 문자열이 ISSUE 나 TIKET로 시작하는경우 ticket-number 라고 판단하여

commit message 하단에 붙혀준다.

![](66690ec2-377c-45df-8c0a-273b6f5ec207.png)

캬아

### pre-commit 로 중요 키 값 커밋 금지

실수로 중요한 키값을 그대로 커밋해서 노출하는것을 hook 로 방지하려한다.

중요 키를 커밋하지 못하게 한다고 했지만

그냥 특정 문자열을 비교해서 같은게 있다면 커밋을 못하게 할 뿐이다.

    #!/bin/sh
    
    #ghp_ : github personal access token
    #sk- : chat-GPT API key
    
    # git 디렉토리 밑의 파일은 검사하지 않는다.
    changed_files=$(git diff --cached --name-only --diff-filter=ACM | grep -v "^git/")
    
    echo "$changed_files"
    bad_keywords='ghp_|sk-'
    
    if [ -z "$changed_files" ]; then
      exit 0
    fi
    
    if echo "$changed_files" | xargs grep -q -E "$bad_keywords"; then
        echo "[commit lint error] Found Bad Keywords in the changes. Please remove them before committing."
        echo "$changed_files" | xargs grep -n -E "$bad_keywords"
        exit 1
    fi

여기서는 "ghp\_" 와 "sk-" 라는 문자열이 변경사항에 있으면 커밋을 못하게 하는중이다.

![](4fb16dad-5d06-4271-b089-19addd309041.png)

키야아

전체 코드는 아래에서 확인 가능하다

[https://github.com/birariro/acceptable-convention/tree/master/git](https://github.com/birariro/acceptable-convention/tree/master/git)

 [GitHub - birariro/acceptable-convention: 이정도면 이쁘게 봐줄만한 컨벤션

이정도면 이쁘게 봐줄만한 컨벤션. Contribute to birariro/acceptable-convention development by creating an account on GitHub.

github.com](https://github.com/birariro/acceptable-convention/tree/master/git)