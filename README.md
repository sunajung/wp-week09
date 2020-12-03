# **주소 API 실습** - by suna jung
---
<br>

## **프로젝트 설명**

> Java Script와 비동기 처리 방식에 관한 수업을 듣고 만든 실습 페이지입니다.<br>
> https://www.juso.go.kr/ 에서 제공하는 주소 API와 https://home.openweathermap.org/에서 제공하는 날씨 API를 사용했습니다. <br> 
> 보내는 사람, 받는 사람, 주소, 내용 입력 후 보내기를 누르면 offline으로 편지가 배송되는 사이트 입니다. <br>
> 주요 코드는 다음과 같습니다. <br>
> - `주소 API 적용` : 주소의 빈칸이나 돋보기를 클릭하면 주소 찾기 팝업창이 뜹니다. 원하는 주소를 입력하고, 선택하면 기존 칸에 적용됩니다.
> - `주소- 페이지네이션 적용` : 주소 찾기 팝업 창에서, 검색 결과가 여러개일 경우 페이지가 나누어 제공됩니다.
> - `날씨 API 적용` : 실시간 날씨와 온도를 우측 하단에 표기해놓았습니다. 날씨에 따라 배송비가 달라집니다.
> - `UI 개선 (CSS)` : CSS 기능을 적절히 활용하여 화면을 구성했습니다.
> - `JS로 정보처리 ` : 각 입력을 받아 저장하고, 처리합니다.

<br>

## **주요 코드 설명** 
---
<br>

> 주요 코드를 설명해드리겠습니다.<br>
> 크게 **`주소 API 적용, 주소- 페이지네이션 적용, 날씨 API 적용, UI 개선 (CSS), JS로 정보처리`** 로 구성됩니다. <br>

### **[ 주소 API 적용 ]**

<br>

- 검색버튼과 주소창 둘 중 어느것을 누르더라도 팝업을 띄우게 하기 위해 변수 2개 (receiverAddressSearchButton, receiverAddressText) 사용, 콜백 등록

``` javascript
 receiverAddressSearchButton.addEventListener('click', receiverPopupFunction);   //검색버튼에 클릭 시 발동하는 콜백(팝업 함수) 등록
 receiverAddressText.addEventListener('click', receiverPopupFunction);      //주소 입력창에도 클릭 시 발동하는 콜백 등록
```

- 팝업에서 주소를 입력받는 부분입니다.

``` javascript
    const form = document.querySelector('#form');   //폼 가져옴
   window.addEventListener('message', function(e) {   //주소 입력 팝업에서 정보가 들어왔을 경우
      if(e.data['child']=='addressPopup'){      //주소 입력 팝업인지 확인.
         form.receiverAddress.value = e.data['content'];         //맞으면 받은 정보를 주소입력창에 대입
      }
   });
```

- API를 통해서 주소검색 결과를 얻어온 후, 총 개수와 안내 메시지 출력

``` javascript
	var selectedPage=null;
    form.addEventListener('submit', async e => {
      e.preventDefault();
      keyword = encodeURIComponent(document.querySelector('#address').value);
      try {
		nowPage = 1;	//검색 시 현재 페이지 1로 초기화
        const response = await search(keyword);
        const txt = await response.text();
        const results = JSON.parse(txt.replace(/^\(/, '').replace(/\)$/, '')).results;
		totalPage = results.common.totalCount;	//https://www.juso.go.kr/addrlink/devAddrLinkRequestGuide.do?menu=roadApi 참고함. result.common.totalCount는 검색 결과 총 개수 나타냄
		searchResult.innerHTML = '검색어 "'+document.querySelector('#address').value+'"에 대한 검색 결과 총 "'+numToString(totalPage)+'"건 입니다.';	//검색 결과 안내 메시지 설정
        display(results);
		this.address.value="";
		pagenation();
      } catch (e) {
        console.error(e);
      }
    });
```

### **[ 주소- 페이지네이션 적용 ]**

<br>

- 현재 페이지와 총 페이지 수를 변수로 저장해 둔 뒤, 안내 메시지 출력 부에서 페이지를 계산합니다.
- 먼저 총 주소 수를 받아오고, 이를 기반으로 표현해야하는 총 페이지 수를 계산합니다. 
  한 화면에 10페이지씩 나오게 하며 한 페이지에 10개의 게시물이 올라오게 제작했습니다.

``` javascript
var nowPage = 0;	//현재 출력해야 하는 페이지
	var totalPage = 0;	//검색 결과 총 페이지 수 ex)검색결과 51개 = 6페이지. 1339개 = 134페이지. 계산은 위에서.
	var keyword;	//검색어
    const form = document.querySelector('#form');	
    const searchResult = document.querySelector('#result');	//검색결과 안내 메시지 부분
	function numToString(number){		//숫자 자동으로 점찍기. ex) 30000 -> 30,000	
		return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");

   const pageSection = document.querySelector('#pagenation-cover'); /* 페이지 번호들 들어갈 공간 */
	function pagenation(){
		pageSection.innerHTML = '';	//초기화
		if(nowPage>10){	//현재 페이지가 10보다 작음 = 이전 페이지로 못감
			var prevPages = document.createElement('div');
			prevPages.innerHTML = '<';
			prevPages.addEventListener('click',function(e){
				nowPage -= 10;	//이전페이지 버튼 누를 경우 10페이지 이전으로 이동. 목록도 10페이지 전 목록
				pagenation(); 	//ex) 현재페이지가 16이면 11~20이 나온 상황. 이전페이지 누르면 현재페이지는 6이 되고 목록은 1~10이 나옴
			});
			pageSection.appendChild(prevPages);	//이전페이지 등록
		}
		else{		//이전페이지 버튼 없으면 자리 맞추기 위해 빈칸 등록
			var prevPages = document.createElement('div');
			prevPages.innerHTML = '　';	//ㄱ한자1 사용. 스페이스바 입력시 깨짐. 원인 알 수 없음.
			pageSection.appendChild(prevPages);		
		}
		
		startPage = nowPage-(nowPage-1)%10	//ex) now = 63 => start = 61, now 29 => start 21.
		for(var i=0;i<10;i++){	//10개 페이지씩 보여 줄 예정
			var page = startPage + i;
			var pageButton = document.createElement('div');
			if(i==0){
				searchAndDisplay(page);	//화살표로 페이지들 이동시마다 각 1번페이지 목록 보여줌. ex) 현재 5페이지. 다음화살표 -> 현재 11페이지.
				pageButton.id = "selected-page";
				selectedPage = pageButton;
			}
			pageButton.innerHTML = page;	//페이지 번호 저장
			pageButton.pageNum = page;		//페이지 번호를 임의로 만든 pageNum변수에 저장
			pageButton.addEventListener('click', pageButtonCallback); //각 페이지 마다 클릭하면 해당하는 페이지의 목록을 가져오는 콜백 등록. 63누르면 63페이지 목록 띄우도록.
			pageSection.appendChild(pageButton);	//페이지 번호 등록
			if(pageButton.pageNum == Math.floor((totalPage-1)/10)+1)return;	//최대 페이지 넘어가면 그대로 종료.
		}
		
		if((Math.floor( (totalPage - 1)/10) + 1) % 10 != 0){	//총 페이지 수가 10으로 나누어 떨어지는 경우 다음 페이지 없음.
			var nextPages = document.createElement('div');
			nextPages.innerHTML = '>';
			nextPages.addEventListener('click',function(e){
				pageSection.innerHTML = '';
				nowPage += 10; //다음페이지들 버튼 누를 경우 10페이지 다음으로 이동. 목록도 10페이지 다음 목록
				pagenation();
			});
			pageSection.appendChild(nextPages);
		}
	}
```

<br><br>

### **[ 날씨 API 적용 ]**

<br>

- https://home.openweathermap.org/ 에서 오늘 날씨와 온도를 받아옵니다. 주소를 받아오는 예제를 응용하여 사용했습니다.

``` javascript
 const kyoungkidoCode = "1841610"; // 경기도 지역 코드. https://openweathermap.org/current에 현재날씨 api 사용법과 각종 code들 있음.
 const weatherAPIKey = "API KEY";   //회원가입하여 기본적으로 제공된 api key
   var API_URL = "http://api.openweathermap.org/data/2.5/weather?id="+kyoungkidoCode+"&appid="+weatherAPIKey;
   
   /* fetch 이용하여 API 호출 */
   fetch(API_URL)
   .then(async function(result){      //결과 받아왔으면
      var txt = await result.text();
      var res=JSON.parse(txt.replace(/^\(/, '').replace(/\)$/, ''));   //문자열을 json으로 바꿔줌. 
      
      nowWeather = res.weather[0].main;    //현재 날씨 받아오기      
      var letterPrice = document.querySelector('#letter-price');   //현재 날씨에 따른 가격 설정
      letterPrice.innerHTML = '현재 배송비는 '+getPrice()+'원 입니다.';
      var weatherIcon = document.querySelector('#weather-icon');   //현재 날씨에 따른 아이콘 설정. API와 함께 아이콘 url도 제공됨. https://openweathermap.org/weather-conditions
      weatherIcon.src = "http://openweathermap.org/img/wn/" + res.weather[0].icon + ".png";
      var weatherName = document.querySelector('#weather-name');   //현재 날씨를 한글로 바꿔 설정
      weatherName.innerHTML = "현재 날씨 : " + weatherNames[nowWeather]+"("+(res.main.temp-273).toFixed(1)+"˚C)";
   });
   
```

<br><br>

### **[ UI 개선 (CSS) ]**

<br>

- 수업시간에 배운 CSS들을 활용하여 마우스 hover시 커서, 색바뀜, 기본 배치등을 구성했습니다.


<br><br>

### **[ JS로 정보처리 ]**

<br>

- 편지 항목 중 하나라도 빈칸이 있으면 보내지 못하도록, switch 문을 이용했습니다.
- 또한 수업시간에 배운 trim을 활용하여 공백만 있는 경우에도 보내지지 않도록 설정했습니다.

``` javascript
      var checkFormReturnValue = checkForm();      //checkForm은 빈 곳이 없으면 0을, 빈 자리가 있으면 해당 자리를 return 함.
      if(checkFormReturnValue != 0){   //빈곳이 있다면
         var content;
         switch(checkFormReturnValue){   //그에 맞는 자리 이름 설정
            case 1:content='보내는 사람을';break;
            case 2:content='받는 사람을';break;
            case 3:content='받는 사람의 주소를';break;
            case 4:content='받는 사람의 상세주소를';break;
            case 5:content='편지 내용을';break;
         }
         alert(content+" 을 입력해주세요.");   //경고메시지
         return false;   //빈칸이 있으므로 이후 행동 하지 않음
      }

	     function checkForm(){
      if(form.sender.value.trim()=="")return 1;   //공백만 써있는 경우도 배제하기 위해 문자열에서 공백을 지우는 trim() 이용.
      if(form.receiver.value.trim()=="")return 2;   //trim이 없으면 띄어쓰기만 있는 입력도 빈칸이 아니라고 생각하고 출력하게 됨
      if(form.receiverAddress.value.trim()=="")return 3;   //trim으로 공백 없애주면 정상적으로 필터링 가능
      if(form.receiverAddressDetail.value.trim()=="")return 4;
      if(form.content.value.trim()=="")return 5;
      
      return 0;
   }
```
<br><br>

- 편지를 보냈다는 alert를 띄웁니다. 이 때 홈페이지에서 받아온 정보를 반영하고, 배송일은 현재 날짜로부터 항상 달라지게 설정했습니다.
- 편지를 다보내고 나서는, reset() 하여 화면에 쓰여진 내용을 모두 지웁니다.

``` javascript
      var today=new Date();
      today.setDate(today.getDate()+2);   //편지는 이틀 후에 도착하므로 2일 이후를 setting. date에 2를 더해주면 알아서 달, 년까지 계산해줌. 
      alert(this.sender.value+"님이 "+this.receiver.value+"님에게 편지를 전송하셨습니다.\n"+   
         (today.getYear()+1900)+"년 "+(today.getMonth()+1)+"월 "+today.getDate()+"일에 도착 예정입니다.\n"+
         "이용료 "+getPrice()+"원이 자동 결제됩니다.");

	this.reset();
```
<br><br>


## **비고 및 고찰** 
---
<br>
 - 여태까지 javascript강의를 4주차에 걸쳐서 수업을 들었는데, 한번 듣고 이해되지 않는 부분이 많아서 과제할 때 시간이 많이 걸렸습니다. 다른 API도 적용해보고 싶고, 편지 기능도 제대로 살리고 싶어서 이것저것 찾아봤는데 노력에 비해 결과물이 크지 않은 것 같아 조금 아쉽습니다. 그래도, 배운 내용을 최대한 다 적용해서 주소 API 예제를 실습해봤습니다! <br>
 - 강의만 들을 때는 잘 이해가 되지 않았는데, 과제를 진행하면서 비동기 처리 방식에 대해 개념이 잡히는 것 같습니다. 아직 복잡한 처리를 하긴 어렵지만 추후 더 연습하면서 실력을 쌓아야겠다고 다짐했습니다.<br>
 - 팝업창을 띄워서 처리하는 부분에서 아예 팝업 API를 사용하려고 시도해봤는데, 서버에서 사용하는것을 전제로 하고있어서 통신을 하기 위한 서버를 구축해야하고, 테스트하는 부분도 어려웠습니다. 그래서 이 부분은 포기하고 직접 구현을 했는데 나중에 큰 프로젝트를 진행하게 된다면 api를 그대로 가져와서 써야겠다고 생각했습니다.  <br><br>

### **제작 시 주요사항** <br>

**1. 페이지 네이션** <br>
=> pagenation함수를 어떻게 구성해야할지 감이 안왔습니다. 그래서 단계별로 처리했습니다. 첫번째로 총 주소수를 받아오고, 지금 화면에서 몇 페이지부터 몇 페이지까지 표시해야하는지를 계산했습니다. 다음 페이지나 이전페이지를 누르면 10단위로 이동하여 표시되게 하고, nowPage는 화면에 표시된 페이지 중 가장 빠른것을 가리키게 만들었습니다. search와 display하는 함수를 searchAndDisplay로 묶어 코드의 가독성을 살리려 노력했습니다.

**2. API 사용** <br>
=> Api사용이 낯설어서 어떻게 정보를 받아오는지 구조를 이해하기 어려웠고, 또 어떤 정보를 받아와서 사용을 어떻게해야하는지 찾는 것 자체에 시간이 오래걸렸습니다. 콘솔창에 띄워보거나 레퍼런스를 참조하면서 조금씩 고칠 수 있었습니다.

**3. 팝업과 현재페이지 사이의 통신.** <br>
=> 팝업간의 통신을 opener로 먼저 시도했었는데, 크롬에서 보안상의 이유로 막혔다고 실행이 되지 않았습니다. 그래서 다른방법-현재 쓰는 postMessage-을 찾느라 많은 시간을 사용했는데, 효과적으로 검색할 수 있는 방법을 익혀야겠다고 깨닫게 되었습니다.

**++ 12시 긴급점검**
=> 갑자기 API기능이 모두 먹통이 됨. 시행착오 끝에 API_URL을 https로 바꾸니 다시 정상적으로 실행됨.


<br><br>
