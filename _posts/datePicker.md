## DatePicker



#### 특정날짜 활성화 / 그외에 disable 하기 

##### - 특정일 선택막기(ex)

```
function disableAllTheseDays(date){
		var m = date.getMonth(), d = date.getDate(), y = date.getFullYear();
			for ( i = 0; I < disabledDays.length; i++){
			if($.inArray(y + '-' + (m+1) + '-' + d, disabledDays) != -1){
					return [false];		//false = 비활성화
			}
		}
		return [true];
	}
```

-> 위내용이 구글 검색하면 나오는 특정일 disalbe 해주는 국룰코드이다 

​	하지만 내가하고싶은것은 배열로 특정일만 활성화되고 그외는 disabled 시키는 코드를 짜고싶은것

##### - 특정일은 활성화하고 그외는 disabled 되게하기

```
	var -assignedDates = []; 	// 스케줄담을 배열 
	function disabledExceptTheDays(date) {
		var baseDate = date.getFullYear() + lPad(date.getMonth()+1) + lPad(date.getDate());		//20211104 
		for (I = 0; I < _assignedDates.length; I++) 						// _assignedDates -> ajax로 넣은 스케줄들
			if(baseDate == _assignedDates[I].order_date) {
				return [true];		// true = 활성화
			}
		}
		return [false];

	}
```

##### $("#ID").datepicker() 안에

```
$("#ID").datepicer(){
	beforShowDay: disabledExceptTheDays, 넣기 

	onChangeMonthYear: function (selectedYear, selectedMonth){
		selectSchedAssignedDates(String(selectedYear) + lPad(seelctedMonth));	// lPad(date.getDate()) -> 1일 - 01일로 
	}
}
```


