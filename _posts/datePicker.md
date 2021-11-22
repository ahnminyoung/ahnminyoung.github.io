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

~~~
$("#ID").datepicer(){
	beforShowDay: disabledExceptTheDays, 넣기 

	onChangeMonthYear: function (selectedYear, selectedMonth){
		selectSchedAssignedDates(String(selectedYear) + lPad(seelctedMonth));	// lPad(date.getDate()) -> 1일 - 01일로 
	}
}
~~~

#### 실전코드(JSP)

~~~
var _assignedDates = [];	// 배차확정

$(function(){
	$(".search_area").css("left", "1208px");
	
	selectScheAssignedDates($("#targetDate").val().substring(0, 6));	// 배차확정 날짜 리스트 조회
	
	//조회 시작 날짜
    $("#searchStrtDate").datepicker({
    	beforeShowDay: disabledExceptTheDays,
        changeYear: true
        , changeMonth: true
        , showMonthAfterYear:true
        , monthNames: ['1','2','3','4','5','6','7','8','9','10','11','12']
        , monthNamesShort: ['1','2','3','4','5','6','7','8','9','10','11','12']
        , dateFormat: "yy년 mm월 dd일(DD)"
        , dayNames: [ "일", "월", "화", "수", "목", "금", "토" ]
    	, dayNamesShort: [ "일", "월", "화", "수", "목", "금", "토" ]
        , dayNamesMin: [ "일", "월", "화", "수", "목", "금", "토" ]
    	//, beforeShowDay: available
    	//, beforeShowDay: disableAllTheseDays
        //, maxDate:new Date()
    	, onChangeMonthYear: function (selectedYear, selectedMonth) {
    		selectScheAssignedDates(String(selectedYear) + lPad(selectedMonth));
        
    	}
        , onSelect: function(selectedDate, inst){
        	//선택일 변환(yyyymmdd)
            var selectedYear = inst.selectedYear;
            var selectedMonth = inst.selectedMonth+1;
            var selectedDay = inst.selectedDay;
            
            //alert(selectedDate);
            $("#targetDate").val(selectedYear + lPad(selectedMonth) + lPad(selectedDay));
            
            selectRealTimeCenter('searchCenter', $("#targetDate").val());					// 센터 목록 조회
            setCntrId();																	// 쿠키 값이 존재하면 존재하는 값으로 센터 세팅 
            selectRealTimeOrder('searchOrder', 'searchCenter', $("#targetDate").val()); 	// 선택 센터별 배송 그룹 조회
            selectRealTimeBaseData();														// 기본정보 조회
            selectScheAssignedDates(selectedYear + lPad(selectedMonth));					// 배차확정 존재 날짜 조회
        	
            $("#contractRegionView").trigger("change");
            
        }

		});
});

function selectScheAssignedDates(targetMonth) {
	
	var param = {
		"url"   	: "/schedule/selectScheAssignedDates.do",
		"type"  	: "post",
		"data"		: {'targetMonth' : targetMonth},
		"async" 	: false,
		"callback"	: function(result) {
			if (result != "" && result != undefined) {
				_assignedDates = result.assignedDates;
			}
			/* if (result != "" && result != undefined) {
				var assignedDate = "";
				for(var i = 0; i < result.assignedDates.length; i++) {
					assignedDate = result.assignedDates[i];
					console.log('assignedDate', assignedDate);
				}
			} */
		}
	}
	_comLib.callAjax(param);
}

function disabledExceptTheDays(date) {
	var baseDate = date.getFullYear() + lPad(date.getMonth()+1) + lPad(date.getDate());
	
	for (i = 0; i < _assignedDates.length; i++) {
		if(baseDate == _assignedDates[i].order_date) {
			return [true];
		}
	}
	return [false];
}

~~~

#### 실전코드(Controller + Service)

```
 
    @RequestMapping("/schedule/selectScheAssignedDates.do")
    @ResponseBody
    public ModelAndView selectScheAssignedDates(@SessionAttribute("user") Map<String, Object> user, @RequestParam Map<String, String> param) {
    	ModelAndView mav = new ModelAndView("jsonView");
        logger.debug("selectScheAssignedDates param : " + param.toString());
        
        param.put("comp_id", (String) user.get("comp_id"));
        
        mav.addObject("assignedDates", scheduleService.selectScheAssignedDates(param));
        
        return mav;
    }
}

	public List<Map<String, Object>> selectScheAssignedDates(Map<String, String> param) {
		return comDao.queryForList("com.kt.twin.schedule.selectScheAssignedDates", param);
	}
```

#### 실전코드(mapper)

```
<select id="selectScheAssignedDates" parameterType="hashMap" resultType="hashMap">
    <![CDATA[
    	select order_date
		  from tw_order tor
		 where comp_id = #{comp_id}
		   and optm_stat_cd = '07'
		   and order_date like #{targetMonth} || '%'
    ]]>
    </select>
```



​	

