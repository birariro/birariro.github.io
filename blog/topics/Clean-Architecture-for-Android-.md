# Clean Architecture for Android 


아키텍쳐의 필요성
=========

태초
--

기존의 서버에서 사용하던 MVC 패턴을 Android,IOS 에 적용하며 개발.

초기에는 큰 문제가없었으나 점점 클라이언트의 몸집이 키지며

관심사 분리에 안되기에 발생하는 문제를 직면

아키텍쳐 도입의 장점
-----------

1.  일관적인 코드 작성으로 **유지보수와 협업에 도움**이된다.
2.  생산성 향상
3.  **테스트의 용이성**
4.  개발의 방향성을 잡을수있다.

종류
--

1.  MVC : model + view + controller
    1.  서버든 클라이언트든 일반적으로 접할수있는 아키텍쳐
    2.  하지만 비지니스 로직과 뷰의 관점 분리가 안되기때문에 안드로이드는 MVP, MVVM을 적용
2.  MVP : model + view(view controller) + Presenter
    1.  이곳에서의 뷰는 뷰콜백을 받아 구성하는 구조
    2.  모델과 뷰를 함께 관리하긴하지만 중간 인터페이스를 활용하여 추상화를 하는것이 특징
3.  MVVM : model + view(view controller) + view model
    1.  view 에서는 model의 변화를 구독하거나 옵저빙, 브로드캐스팅을 활용하여 뷰를 변경
4.  MVVM + DataBinding
    1.  기존의 MVVM에서 view가 model을 구독하는 보일러플레이트코드를 바인딩을 통해 제거한다.
5.  MVI : Model + View + Intent
    1.  최근에 유행되는 아키텍쳐로 단방향 플로우의 대표적이다.
    2.  MVP, MVVM 의 생태 제어 한계를 극복하기위한 패턴으로 불변한 상태를 가지는 특징
6.  MvRx(에어비앤비), Flux(페이스북), Rids(우버), ...etc...
    1.  단방향 플로우들로 대표하는 각 회사들이 구성하는 아키텍쳐 패턴 이다.
7.  절망의 Android Clone Architecture
    1.  Rebert C 선생님이 만드신 아키텍처를 3계층으로 분리하고 관심사 또한 분리하여 변경이 쉬운 객체를 의존하지않고 변경에 강한 설계
    2.  많은 기업들은 MVVM + AAC(LiveData, DataBinding, ViewBinding, Room) + Repository 패턴을 통한 구현을 사용.

Clean Architecture for Android
==============================

![](587e64f9-03d0-45b7-a18c-d6afef891d49.png)

가장 유명한 그림으로 위와같이 3개의 계층으로 설계를 진행하며

Presentation 계층은 **Domain** 계층을 의존하고

Data 계층도 **Domain** 계층을 의존하며

**Domain** 계층은 그 어떠한 계층을 의존하지않는다.

**Domain** 계층에서 Data 계층의 Repository를 바로 사용하면 의존성을 가지게되기때문에

인터페이스를 **Domain** 계층에서 가지고

Data계층에서 이를 구현하는것으로 **의존관계를 역전**한다.

사실 구현해보면 위의 그림에서 문제가 있다.

위의 그림으로만 본다면 Data 계층에서 받은 정보를 Domain 계층에서 Tranlater 을 통해

Domain 계층에서 사용할 모델로 변경하게되는데

이는 곧 Domain 계층이 Data계층에 의존성을 가지게되는 문제가 생긴다.

따라서 Tranlater 이 Data계층에서 역활을 수행하며 Domain 계층에서 필요로하는 model 을 의존하여 데이터를 변경해서 내려줘야한다.

즉 Domain 계층은 다른 계층의 의존성을가지면 안된다.

![](1d1d9f94-63d0-4960-b39e-dd3941e8fca5.png)

Model 과 Entity의 차이
------------------

*   model : 비지니스 로직에서 필요로 하는 데이터
*   entity : 서버로 부터 받은 데이터

클린 아키텍쳐의 기대
-----------

### 희망

![](dfc3ffcb-9d7b-4f57-b1b0-db12093785fc.png)

1.  이미 많이 사용하고있으니 충분한 레퍼런스가 있다.
2.  클린한 아키텍쳐 설계로 인해 클린한 코드가 자연스럽게 따라온다.
3.  유지 보수가 용이해진다.
4.  좋은 기술 스택 하나 생기는 걸꺼야

### 그리고 절망

![](4591586c-1f6f-4436-990c-e94fbc8b7a57.png)

1.  레퍼런스는 많은데 다 구현 방식이 다름
2.  어떻게 하라는지 잘 모르겠음
3.  MVI 가 MVVM 보다 좋다고?
4.  코드가 클린해지지만 엄청난 양의 클레스파일들..

![](77a76fd3-a010-4147-bfe8-2b8c2d393d9c.png)

고작 4개의 화면..

필요 라이브러리
========

AAC(Android Architecture Components)
------------------------------------

테스트와 유지보수가 쉬운 앱을 디자인할수있도록 돕는 라이브러리 모음으로

ViewModel, LiveData, Room등을 제공한다.

그리고 이를통해 클린 아키텍쳐를 설계하기 용이해지며

보일러플레이트를 줄일수있다.

DI 도구
-----

직접 구현할수있지만 잘 만들어져있는걸 쓰는건 좋다.

대표적으로 Dagger, Hilt, Koin(유사 DI 도구) 이 존재한다.

좋은 순

쉬운 순

추천 순

Degger, Hilt, Koin

Koin, Hilt, Degger

Hilt, Degger, Koin

각 레이어의 역활 규칙
============

💡공식적인 규칙이 아니며 구현을 해보면서 각 역활을 깨지않고 서로의 역활을 충실히 하기위해서 필요한 룰이라고 생각되는것임을 알립니다.

View
----

1.  사용자의 이벤트를 받는다.
2.  이벤트를 직접 처리하지않고 ViewModel 에게 전달한다.
    1.  클릭, 텍스트 입력 등
3.  ViewModel 이 제공하는 LiveData를 감시하면서 다음 액션을 취한다.
    1.  화면이동, Toast 메시지
4.  Activity/Fragment 안에 if,for 등 로직이 동작하면 안된다.(강남 언니 블로그)
    1.  if 는 봐줄 수 있지않을까?

    class LoginActivity : AppCompatActivity() {
        private lateinit var binding :ActivityLoginBinding
        private val viewModel : LoginViewModel by viewModels()
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            binding = ActivityLoginBinding.inflate(layoutInflater)
            setContentView(binding.root)
      
    			  val id = binding.idEditText
            val pw = binding.pwEditText
    
            id.afterTextChanged {
                viewModel.validLoginData(id.text.toString(),pw.text.toString())
            }
            pw.afterTextChanged {
                viewModel.validLoginData(id.text.toString(),pw.text.toString())
            }
            binding.bottomLayout.resultBtn.setOnClickListener {
                binding.loading.visibility = View.VISIBLE
                viewModel.login(id.text.toString(),pw.text.toString())
    
            }
    				viewModel.loginFormState.observe(this, Observer{
                //로그인을 할수있도록 모든 입력이 되었는지 감지
            })
    
            viewModel.loginResult.observe(this, Observer {
                //로그인을 클릭했다면 결과는 어떤지 감지
            })
        }
       
    }
    

ViewModel
---------

1.  사용자에게 발생되는 모든 이벤트를 받을수있는 메소드를 View에게 제공한다.
2.  View가 액션을 취할수있는 LiveData를 제공한다.
3.  그 외의 기능은 제공하지않는다.(비지니스 로직을 가지고있으면 안된다.)
4.  Activity/Fragment는 여러 ViewModel을 가지지않는것이 권장 (구글)
5.  ViewModel 에서 **Context**, **LifecycleOwner** 등 사용하는 모든 행위를 하면 안된다.(옵저빙, 내부 저장소 접근 등)

    @HiltViewModel
    class LoginViewModel @Inject constructor(
        private val loginUseCase: LoginUseCase,
        private val validDataUseCase: ValidDataUseCase
        ) :ViewModel() {
     
        //로그인 입력 폼 valid 
        private val _loginFormState = MutableLiveData<LoginFormStateData>()
        val loginFormState: LiveData<LoginFormStateData> = _loginFormState
    
        //로그인 결과 
        private val _loginResult = MutableLiveData<CommonResultData>()
        val commonResult :LiveData<CommonResultData> = _loginResult
    
        fun login(id:String, pw:String){
            val validLoginData = validIDAndPWD(id,pw)
            if(!validLoginData.validID || !validLoginData.validPW){
                _loginResult.value = CommonResultData(success = false,errorCode = -1)
                return
            }
            val loginData = LoginData(id =id, pw = pw)
    
            viewModelScope.launch {
                val loginResultFlow = loginUseCase.execute(loginData)
                loginResultFlow.collect {
                    _loginResult.value = it
                }
            }
    
        }
    
        fun validLoginData(id:String,pw:String){
            _loginFormState.value = validIDAndPWD(id,pw)
        }
        
        private fun validIDAndPWD(id:String, pw:String) : LoginFormStateData {
            if(!validDataUseCase.execute(ValidDataUseCase.ValidDataType.id,id)){
                return LoginFormStateData(validID = false, validPW = false)
            }else if(!validDataUseCase.execute(ValidDataUseCase.ValidDataType.password,pw)){
                return LoginFormStateData(validID = true, validPW = false)
            }
            return LoginFormStateData(validID = true, validPW = true)
    
        }
    }
    

UseCase
-------

1.  하나의 UseCase는 실행가능한 한개의 Public 메소드만을 제공한다.
2.  업무의 요구사항을 담고있으며 입력,결과물을 생성하기위한 처리를 기술한다.
3.  플렛폼의 의존성 없이 수행되어야한다.(라이브러리화 가능해야한다)
4.  개발지식이없는 사람에게 UseCase리스트를 보여주면 해당 Domain이 무슨 업무를 수행하는지 이해할수있어야한다. 
5.  UseCase는 다른 UseCase를 주입받아 사용해도된다.
    1.  댓글 UseCase에서 이미지 URL을 얻어오는 UseCase가 필요할때 이를 주입받아 사용해도된다.
    2.  이때 댓글 UseCase는 High Level UseCase라고 한다.
    3.  이때 이미지 URL UseCase는 Low Level UseCase라고한다.
    4.  Low는 High 를 참조하면 안된다. (순환 이슈 발생)

    class LoginUseCase (private val memberRepository: MemberRepository){
        suspend fun execute(id:String, pw:String):String{
            return memberRepository.login(id,pw)
        }
    }
    class FindIDUseCase (private val memberRepository: MemberRepository){
        suspend fun execute():String{
            return memberRepository.find()
        }
    }
    

Model
-----

UseCase 에서 필요로 하거나 ViewModel 에서 필요로 하는 데이터 형식이다.

서버에서 내려주는 데이터를 그대로 쓰는경우는 사실 모바일을 기준으로 비지니스를 설계하는 경우는 가능하지만

그렇지않은 경우가 많다.

모바일에서 필요로 하는 데이터 형식을 가진다.

    data class LoginResultData(
        val success: Boolean = false,
        val errorCode: Int = -1
    )
    

Domain 계층의 Repository
---------------------

클린 아키텍쳐의 핵심인 Domain 계층은 Data 계층의 존재를 알지못해야한다.

로버트 마틴의 DIP 원칙에 따르면

> 자기보다 변하기 쉬운 것에 의존하지 마라

라는 원칙에 의해

Domain 계층은 인터페이스를 의존하고

Data계층은 인터페이스를 구현하는것으로 의존성을 역전한다.

Data 계층의 Repository
-------------------

Domain 의 인터페이스를 구현한다.

Repository 의 중요 역활은 데이터의 출처와 관계없이 동일한 인터페이스로 데이터에 접속할수있게 만드는것으로

DataSource 혹은 DataStore 에 접근하는 기능을 캡슐화하여

Domain 계층에서 데이터를 얻어갈때 이 데이터가 Remote 에서 얻은것인지

Local 에서 얻은것인지 알필요 없도록한다.

    class CommonMemberRepository (
        private val remoteDataSource: RemoteDataSource,
        private val localDataSource: LocalDataSource,
        private val fireBaseDataSource: FireBaseDataSource):MemberRepository
    {
        override suspend fun signup(loginData: LoginData) : Flow<CommonResultData> {
            return fireBaseDataSource.signup(loginData.id,loginData.pw)
        }
    
        override suspend fun login(loginData: LoginData) : Flow<CommonResultData> {
            //return remoteDataSource.login(loginData.id,loginData.pw)
            return fireBaseDataSource.login(loginData.id,loginData.pw)
        }
    
        override suspend fun deleteLoginData() {
            localDataSource.deleteLoginData()
        }
    }
    

DataStore
---------

데이터에 실제 접근하는 로직을 담당한다.

Tranlater
---------

DataStore 에서 데이터를 얻어온후 Domain 이 필요로하는 Model로 변환하는 역활을수행한다.

🤔진행중에 발생한 의문
=============

### ViewModel에서 DataSoruce까지의 깊이가있는데 DataSource 콜백으로 리턴하는경우 콜백이 깊어질텐데 어떻게 할까?

> 이경우 콜백으로도 처리해보고 LiveData로 도 해결해보고 Block로 도 해보았지만 모두 레이어의 룰에 어긋나게 되었다. 지금까지 가장 좋은것은 Flow(Kotlin), RxJava(java) 로 해결이 였다

### Repository 는 단순 DataSoruce 에 연결해주는 맵핑이 아닌가?

> 단순 연결이라 볼수도 있지만 사실상 데이터가 어느 엔드포인트로(remote,local) 을 정해주는 역활만 수행한다고 봐도 될것같다.