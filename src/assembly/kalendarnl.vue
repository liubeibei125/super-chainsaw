<template>
  <div>  
		<div>
			<el-input suffix-icon="el-icon-date" type="text" @focus="getFocus" :value="showTime" class='form-input-150' :class="longInput !== 0 ? 'addBirthday' : 'addLittleBirthday'" placeholder="请选择出生日期" readonly="readonly"></el-input>
		</div>
		
		<el-dialog
			:title="'万年历(' + (switchType/1===1?'公历':'农历') +')'"
			:visible.sync="dia"
			width="600px"
			append-to-body>
      <!-- 阴历页面展示 -->
			<div class = "kalendar" :style="{'z-index': '9999 !important', 'border':switchType / 1 === 2 ? '2px solid #CB1C18' : ''}" v-if="switchType/1===2"> 
				<div class="result-op c-container xpath-log" srcid="4254" id="1" tpl="calendar_new" mu="http://nourl.baidu.com/6018" data-op="{'y':'CFBFFBFF'}" data-click="{&quot;p1&quot;:&quot;1&quot;,&quot;rsv_bdr&quot;:&quot;0&quot;,&quot;fm&quot;:&quot;alop&quot;,&quot;rsv_stl&quot;:&quot;0&quot;,&quot;p5&quot;:1}"> 
					<div class="op-calendar-new">
						<div class="op-calendar-new-box">
						<div class="op-calendar-new-left c-clearfix"> 
							<div class="op-calendar-new-select-box topSelect" style="visibility: visible;"> 
								<div class ="year" style="margin-right: 10px;">
									<el-select v-model="nlyear" filterable size="mini" class ="selectYearAndMonth" @change="handleNlYearChange" :loading="addressYearLoading" placeholder="年" style="width: 80px">
										<el-option
											width="80%"
											v-for="item in 2020"
											v-if="item > 1899"
											:key="item"
											:label="item + '年'"
											:value="item">
										</el-option>
									</el-select>
								</div>
								<div class ="nlmonth month"  style="margin-right: 10px;">
									<span class ="el-icon-arrow-left" @click="prevNlMonth()"></span>          
									<el-select size="mini" filterable v-model="nlmonth" class ="selectYearAndMonth"  @change="handleNlMonthChange" :loading="addressMonthLoading" placeholder="月" style="width: 60px">
										<el-option
											v-for="(item, index) in monthArr"
											:key="index"
											:label="item"
											:value="index + 1">
										</el-option>
									</el-select>
									<span class = "el-icon-arrow-right" @click="nextNlMonth()"></span>          
								</div>
								<div class ="time">
                  <el-select v-model="hour" size="mini" class ="selectYearAndMonth" popper-class="kalendarHour" :loading="addressYearLoading" placeholder="请选择时辰" style="width: 140px;">
										<el-option v-for="item in dateTime" :label="item.label" :value="item.value" :key="item.value"></el-option>
									</el-select>
								</div>
							</div> 
							<span class = "toToday">
              <el-button type="primary" hidefocus="true" size="mini" @click="goToNlDay()"  style="background-color: #CB1C18; border: 1px solid #CB1C18;">今天</el-button>
							</span>
							<!-- <a class="c-btn op-calendar-new-backtoday OP_LOG_BTN" hidefocus="true" @click="goToDay()" href="javascript:;">返回今天</a>        -->
							<div class="op-calendar-new-table-box c-gap-top" style="border-top: 1px solid #CB1C18">
							<table class="op-calendar-new-table op-calendar-new-table-six noTopBorder">
								<tbody>
								<tr>
									<th class="op-calendar-new-table-weekend">日</th>
									<th>一</th>
									<th>二</th>
									<th>三</th>
									<th>四</th>
									<th>五</th>
									<th class="op-calendar-new-table-weekend">六</th>
								</tr>
								<tr v-for="(oneWeeks,indexa) in nlDayForArr" v-if="!(indexa===5 && oneWeeks[0].lunarMonth!==nlmonth)">
										<td v-for="(item,index) in oneWeeks">
											<div class="op-calendar-new-relative"> 
												<a href="javascript:;"  hidefocus="true" :class="[(index===0 || index===6)?'op-calendar-new-table-weekend':'',nlmonth!=item.lunarMonth?'op-calendar-new-table-other-month':'',(nlday===item.lunarDay && nlmonth===item.lunarMonth)?'op-calendar-new-table-festival op-calendar-new-table-selected':'']" @click="chooseNlDay(item.lunarYear, item.lunarMonth, item.lunarDay, false)"> <span class="op-calendar-new-daynumber">{{item.lunarDayName}}</span> <span class="op-calendar-new-table-almanac">{{ item.gl_day===1?item.gl_month+'月':item.gl_day+'/'+item.gl_month+'月' }}</span> <span class="op-calendar-new-table-ticket "></span> </a> 
											</div> 
										</td>
								</tr>
								</tbody>
							</table>
							</div>
						</div> 
						<div class="op-calendar-new-right yin" :style="{'height':leftHight, 'background':switchType / 1 === 2 ? 'linear-gradient(-200deg,orange,#CB1C18)' : ''}">
						<!-- <div class="op-calendar-new-right yin" :style="{'height':leftHight, 'background':switchType / 1 === 2 ? 'red' : ''}"> -->
              <div class="changeKalendar">
                <el-radio-group v-model="switchType" size="small" @change="changeSwitch">
                  <el-radio-button label="1">公历</el-radio-button>
                  <el-radio-button label="2" radio-class = "hah" style="background-color: #CB1C18;">农历</el-radio-button>
                </el-radio-group>
              </div>
							<p class="op-calendar-new-right-date">{{ dayInfo.lunarYear+'年 '+dayInfo.lunarMonthName+dayInfo.lunarDayName }}</p> 
							<p class="op-calendar-new-right-day nlDay">{{dayInfo.lunarDayName}}</p> 
							<p class="op-calendar-new-right-lunar c-gap-top-small"> <span>{{ year+'-'+(month<=9?'0'+month:month)+'-'+(day<=9?'0'+day:day) }} {{ weekDay }}</span><span>{{ dayInfo.GanZhiYear }}年 【{{ dayInfo.zodiac }}年】</span><span>{{ dayInfo.GanZhiMonth }}月 {{ dayInfo.GanZhiDay }}日</span> </p>
						</div> 
						</div> 
						<div class="op-calendar-new-holidaytip" style="display: none;">
						<span class="op-calendar-new-holidaytip-icon">○<i>!</i></span> 
						<p>10月1日至7日放假调休，共7天。</p> 
						<p>拼假建议：2018年9月25日（周二）~2018年9月30日（周日）请假6天，与中秋节衔接，拼16天小长假</p> 
						</div> 
					</div> 
					<div>
						<span class="c-showurl"> </span>
					</div> 
				</div>   
			</div>
      <!-- 公历页面展示 -->
      <div class = "kalendar" :style="{'z-index': '9999 !important', 'border':switchType / 1 === 1 ? '2px solid #57abff' : ''}" v-if="switchType/1===1"> 
        <div class="result-op c-container xpath-log" srcid="4254" id="1" tpl="calendar_new" mu="http://nourl.baidu.com/6018" data-op="{'y':'CFBFFBFF'}" data-click="{&quot;p1&quot;:&quot;1&quot;,&quot;rsv_bdr&quot;:&quot;0&quot;,&quot;fm&quot;:&quot;alop&quot;,&quot;rsv_stl&quot;:&quot;0&quot;,&quot;p5&quot;:1}"> 
          <div class="op-calendar-new">
            <div class="op-calendar-new-box">
            <div class="op-calendar-new-left c-clearfix"> 
              <div class="op-calendar-new-select-box topSelect" style="visibility: visible;"> 
                <div class ="year" style="margin-right: 10px;">
                  <el-select v-model="year" filterable size="mini" class ="selectYearAndMonth" @change="handleYearChange" :loading="addressYearLoading" placeholder="年" style="width: 80px">
                    <el-option
                      width="80%"
                      v-for="item in 2020"
                      v-if="item > 1900"
                      :key="item"
                      :label="item + '年'"
                      :value="item">
                    </el-option>
                  </el-select>
                </div>
                <div class ="month"  style="margin-right: 10px;">
                  <span class ="el-icon-arrow-left" @click="prevMonth()"></span>          
                  <el-select size="mini" filterable v-model="month" class ="selectYearAndMonth"  @change="handleMonthChange" :loading="addressMonthLoading" placeholder="月" style="width: 60px">
                    <el-option
                      v-for="item in 12"
                      :key="item"
                      :label="item + '月'"
                      :value="item">
                    </el-option>
                  </el-select>
                  <span class = "el-icon-arrow-right" @click="nextMonth()"></span>          
                </div>
                <div class ="time">
                  <el-select v-model="hour" size="mini" class ="selectYearAndMonth" popper-class="kalendarHour" :loading="addressYearLoading" placeholder="请选择时辰" style="width: 140px;">
                    <el-option v-for="item in dateTime" :label="item.label" :value="item.value" :key="item.value"></el-option>
                  </el-select>
                </div>
              </div> 
              <span class = "toToday">
                <el-button type="primary" hidefocus="true" size="mini" @click="goToDay()">今天</el-button>
              </span>
              <!-- <a class="c-btn op-calendar-new-backtoday OP_LOG_BTN" hidefocus="true" @click="goToDay()" href="javascript:;">返回今天</a>        -->
              <div class="op-calendar-new-table-box c-gap-top">
              <table class="op-calendar-new-table op-calendar-new-table-six">
                <tbody>
                <tr>
                  <th class="op-calendar-new-table-weekend glTextColor">日</th>
                  <th>一</th>
                  <th>二</th>
                  <th>三</th>
                  <th>四</th>
                  <th>五</th>
                  <th class="op-calendar-new-table-weekend glTextColor">六</th>
                </tr>
                <tr v-for="(oneWeeks,indexa) in dayForArr" v-if="!(indexa===5 && oneWeeks[0].month!==month)">
                    <td v-for="(item,index) in oneWeeks">
                      <div class="op-calendar-new-relative"> 
                        <a href="javascript:;"  hidefocus="true" :class="[(index===0 || index===6)?'op-calendar-new-table-weekend glTextColor':'',month!=item.month?'op-calendar-new-table-other-month':'',(day===item.day && month===item.month)?'op-calendar-new-table-festival op-calendar-new-table-selected':'']" @click="chooseDay(item.year, item.month, item.day, false)"> <span :class="index === 0 || index === 6 && month===item.month? 'op-calendar-new-daynumber glTextColor' : 'op-calendar-new-daynumber'">{{item.day}}</span> <span :class="day===item.day ? 'op-calendar-new-table-almanac glTextColor' : 'op-calendar-new-table-almanac'">{{ item.lunarDay===1?item.lunarMonthName:item.lunarDayName }}</span> <span class="op-calendar-new-table-ticket"></span> </a> 
                      </div> 
                    </td>
                </tr>
                </tbody>
              </table>
              </div>
            </div> 
            <div class="op-calendar-new-right" :style="{'height':leftHight}">
              <div class="changeKalendar">
                <el-radio-group v-model="switchType" size="small" @change="changeSwitch">
                  <el-radio-button label="1">公历</el-radio-button>
                  <el-radio-button label="2">农历</el-radio-button>
                </el-radio-group>
              </div>
              <p class="op-calendar-new-right-date">{{ year+'-'+(month<=9?'0'+month:month)+'-'+(day<=9?'0'+day:day) }} {{ weekDay }}</p> 
              <p class="op-calendar-new-right-day">{{ day<=9?'0'+day:day }}</p> 
              <p class="op-calendar-new-right-lunar c-gap-top-small"> <span>{{ dayInfo.lunarYear+' '+dayInfo.lunarMonthName+dayInfo.lunarDayName }}</span><span>{{ dayInfo.GanZhiYear }}年 【{{ dayInfo.zodiac }}年】</span><span>{{ dayInfo.GanZhiMonth }}月 {{ dayInfo.GanZhiDay }}日</span> </p>
            </div> 
            </div> 
            <div class="op-calendar-new-holidaytip" style="display: none;">
            <span class="op-calendar-new-holidaytip-icon">○<i>!</i></span> 
            <p>10月1日至7日放假调休，共7天。</p> 
            <p>拼假建议：2018年9月25日（周二）~2018年9月30日（周日）请假6天，与中秋节衔接，拼16天小长假</p> 
            </div> 
          </div> 
          <div>
            <span class="c-showurl"> </span>
          </div> 
        </div>   
      </div>
			<span slot="footer" class="dialog-footer">
				<el-button @click="dia = false">取 消</el-button>
				<el-button type="primary" @click="confirmDay">确 定</el-button>
			</span>
		</el-dialog>
  </div>

  
</template>
<script>
  import LunarCalendar from 'lunar-calendar'
  export default {
    props: {
      dateArr: {
        type: Array,
        default: function () {
          var date = new Date()
          var year = date.getFullYear()
          var month = date.getMonth() + 1
          var day = date.getDate()
          var hour = date.getHours()
          return [year, month, day, hour]
        }
      },
      dateType: {
        type: Number,
        default: 1
      },
      longInput: ''
    },
    data () {
      return {
        showTime: '',
        switchType: '',
        monthArr: [],
        nlDayForArr: [],
        dayArr: [],
        dayForArr: [],
        year: '',
        month: '',
        day: '',
        nlyear: '',
        nlmonth: '',
        nlday: '',
        dayInfo: [],
        addressYearLoading: false,
        addressMonthLoading: false,
        dateTime: [
          {
            label: '时辰未知',
            hourName: '时辰未知',
            value: -1
          },
          {
            label: '早子(0:00-0:59)',
            hourName: '子时',
            value: 0
          },
          {
            label: '丑(1:00-1:59)',
            hourName: '丑时',
            value: 1
          },
          {
            label: '丑(2:00-2:59)',
            hourName: '丑时',
            value: 2
          },
          {
            label: '寅(3:00-3:59)',
            hourName: '寅时',
            value: 3
          },
          {
            label: '寅(4:00-4:59)',
            hourName: '寅时',
            value: 4
          },
          {
            label: '卯(5:00-5:59)',
            hourName: '卯时',
            value: 5
          },
          {
            label: '卯(6:00-6:59)',
            hourName: '卯时',
            value: 6
          },
          {
            label: '辰(7:00-7:59)',
            hourName: '辰时',
            value: 7
          },
          {
            label: '辰(8:00-8:59)',
            hourName: '辰时',
            value: 8
          },
          {
            label: '巳(9:00-9:59)',
            hourName: '巳时',
            value: 9
          },
          {
            label: '巳(10:00-10:59)',
            hourName: '巳时',
            value: 10
          },
          {
            label: '午(11:00-11:59)',
            hourName: '午时',
            value: 11
          },
          {
            label: '午(12:00-12:59)',
            hourName: '午时',
            value: 12
          },
          {
            label: '未(13:00-13:59)',
            hourName: '未时',
            value: 13
          },
          {
            label: '未(14:00-14:59)',
            hourName: '未时',
            value: 14
          },
          {
            label: '申(15:00-15:59)',
            hourName: '申时',
            value: 15
          },
          {
            label: '申(16:00-16:59)',
            hourName: '申时',
            value: 16
          },
          {
            label: '酉(17:00-17:59)',
            hourName: '酉时',
            value: 17
          },
          {
            label: '酉(18:00-18:59)',
            hourName: '酉时',
            value: 18
          },
          {
            label: '戌(19:00-19:59)',
            hourName: '戌时',
            value: 19
          },
          {
            label: '戌(20:00-20:59)',
            hourName: '戌时',
            value: 20
          },
          {
            label: '亥(21:00-21:59))',
            hourName: '亥时',
            value: 21
          },
          {
            label: '亥(22:00-22:59)',
            hourName: '亥时',
            value: 22
          },
          {
            label: '晚子(23:00-23:59)',
            hourName: '子时',
            value: 23
          }
        ],
        hour: '',
        dia: false,
        leftHight: '430px'
      }
    },
    methods: {
      getFocus () {
        this.dia = true
      },
      cancel () {
        this.dia = false
      },
      // 切换公历阴历函数
      changeSwitch (value) {
        var dateArr = [this.year, this.month, this.day, this.hour]
        if (value / 1 === 1) {
          this.changeDay(dateArr, true)
          return
        }
        this.changeNlDay(dateArr, true)
      },
      // 确定日期函数，响应给父组件数据 公历阴历通用
      confirmDay () {
        this.dia = false
        let dayInfo = this.dayInfo
        var calendarData = {}
        calendarData.gl_year = this.year
        calendarData.gl_month = this.month <= 9 ? '0' + this.month : this.month
        calendarData.gl_day = this.day <= 9 ? '0' + this.day : this.day
        calendarData.date = this.year + '-' + calendarData.gl_month + '-' + calendarData.gl_day
        calendarData.hour = this.hour
        calendarData.sx = dayInfo.zodiac
        calendarData.nl_year_name = dayInfo.GanZhiYear
        calendarData.nl_month_name = dayInfo.lunarMonthName
        calendarData.nl_day_name = dayInfo.lunarDayName
        calendarData.nl_ganzhi_month = dayInfo.GanZhiMonth
        calendarData.nl_ganzhi_day = dayInfo.GanZhiDay
        calendarData.nl_run = dayInfo.lunarLeapMonth
        calendarData.showDia = false
        // 获取input展示value
        var hourName = ''
        var hour = this.hour
        this.dateTime.forEach(function (e, i) { // 计算时辰
          if (e.value === hour) {
            hourName = e.hourName
          }
        })
        calendarData.hourName = hourName
        this.showTime = calendarData.date + ' (公历)  /  ' + dayInfo.GanZhiYear + '年 ' + dayInfo.lunarMonthName + ' ' + dayInfo.lunarDayName + ' ' + hourName + ' (农历)'
        this.$emit('changeDay', calendarData)
      },
      // 以下为公历js交互函数
      handleYearChange (year) {
        this.year = year
        let daysObj = LunarCalendar.calendar(this.year, this.month, true)
        this.dayArr = daysObj.monthData
        this.chooseDay(this.year, this.month, this.day, false)
      },
      // 月份选择处理
      handleMonthChange (month) {
        this.month = month
        let daysObj = LunarCalendar.calendar(this.year, this.month, true)
        let monthDays = daysObj.monthDays
        this.dayArr = daysObj.monthData
        if (this.day > monthDays) { // 如果上个月没有31日，选择上个月的30日以此类推
          this.day = monthDays
        }
        this.chooseDay(this.year, this.month, this.day, false)
      },
      // 快速切换上个月
      prevMonth () {
        var month = this.month
        month -= 1
        this.month = month
        if (month <= 0) { // 如果月份小于1 跳到上一年12月
          this.year = this.year - 1
          this.month = 12
        }
        this.handleMonthChange(this.month)
      },
      // 快速切换下个月
      nextMonth () {
        var month = this.month
        month += 1
        this.month = month
        if (month >= 13) { // 如果月份大于12 跳到下一年一月
          this.year = this.year + 1
          this.month = 1
        }
        this.handleMonthChange(this.month)
      },
      // 快速返回今天
      goToDay () {
        var date = new Date()
        var year = date.getFullYear()
        var month = date.getMonth() + 1
        var day = date.getDate()
        var hour = date.getHours()
        this.changeDay([year, month, day, hour], false)
      },
      // 获取某年某月的全部day数据
      getDays (year, month) {
        let daysObj = LunarCalendar.calendar(year, month, true)
        this.dayArr = daysObj.monthData
      },
      // 页面点击日期触发，智能选择本月还是上月
      chooseDay (year, month, day, changeShowTime) {
        var getNewData = false
        if (month !== this.month) {
          this.month = month
          getNewData = true
        }
        if (year !== this.year) {
          this.year = year
          getNewData = true
        }
        if (getNewData) {
          this.getDays(this.year, this.month)
        }
        this.day = day
        this.dayInfo = LunarCalendar.solarToLunar(this.year, this.month, this.day)
        this.nlyear = this.dayInfo.lunarYear
        this.nlmonth = this.dayInfo.lunarMonth
        this.nlday = this.dayInfo.lunarDay
        if (changeShowTime) { // 是否更改input表单value，在页面点击选择日期不触发
          var hourName = ''
          var hour = this.hour
          this.dateTime.forEach(function (e, i) {
            if (e.value === hour) {
              hourName = e.hourName
            }
          })
          this.showTime = this.year + '-' + this.month + '-' + this.day + ' (公历)  /  ' + this.dayInfo.GanZhiYear + '年 ' + this.dayInfo.lunarMonthName + ' ' + this.dayInfo.lunarDayName + ' ' + hourName + ' (农历)'
        }
      },
      // 日期数组改变监听处理函数
      changeDay (dateArr, changeShowTime) {
        if (dateArr.length === 0) {
          var date = new Date()
          this.year = date.getFullYear()
          this.month = date.getMonth() + 1
          this.day = date.getDate()
          this.hour = date.getHours()
          this.getDays(this.year, this.month)
          this.chooseDay(this.year, this.month, this.day, changeShowTime)
          this.showTime = ''
          return
        }
        this.year = dateArr[0] / 1
        this.month = dateArr[1] / 1
        this.day = dateArr[2] / 1
        this.hour = dateArr[3] / 1
        this.getDays(this.year, this.month)
        this.chooseDay(this.year, this.month, this.day, changeShowTime)
      },
      // 以下是农历（阴历）js交互函数
      // 日期数组改变监听处理函数
      changeNlDay (dateArr, changeShowTime) {
        if (dateArr.length === 0) {
          var date = new Date()
          this.year = date.getFullYear()
          this.month = date.getMonth() + 1
          this.day = date.getDate()
          this.hour = date.getHours()
          var lunar = LunarCalendar.solarToLunar(this.year, this.month, this.day)
          this.nlyear = lunar.lunarYear
          this.nlmonth = lunar.lunarMonth
          this.nlday = lunar.lunarDay
          this.handleNlYearChange(this.nlyear)
          this.chooseNlDay(this.nlyear, this.nlmonth, this.nlday, true)
          return
        }
        this.year = dateArr[0] / 1
        this.month = dateArr[1] / 1
        this.day = dateArr[2] / 1
        this.hour = dateArr[3] / 1
        lunar = LunarCalendar.solarToLunar(this.year, this.month, this.day)
        this.nlyear = lunar.lunarYear
        this.nlmonth = lunar.lunarMonth
        this.nlday = lunar.lunarDay
        this.handleNlYearChange(this.nlyear)
        this.chooseNlDay(this.nlyear, this.nlmonth, this.nlday, true)
      },
      // 页面点击日期触发，智能选择本月还是上月
      chooseNlDay (year, month, day, changeShowTime) {
        var getNewData = false
        if (month !== this.nlmonth) {
          this.nlmonth = month
          getNewData = true
        }
        if (year !== this.nlyear) {
          this.nlyear = year
          this.monthArr = this.getMonths(this.nlyear)
          getNewData = true
        }
        if (getNewData) {
          let nlDays = this.getNlDays(this.nlyear, this.nlmonth)
          this.nlDayForArr = nlDays.days
        }
        this.nlday = day
        // 获取dayinfo
        var lunar = LunarCalendar.lunarToSolar(year, month, day)  // 农历转阴历
        this.year = lunar.year
        this.month = lunar.month
        this.day = lunar.day
        this.dayInfo = LunarCalendar.solarToLunar(lunar.year, lunar.month, lunar.day)
        if (changeShowTime) { // 是否更改input表单value，在页面点击选择日期不触发
          var hourName = ''
          var hour = this.hour
          this.dateTime.forEach(function (e, i) {
            if (e.value === hour) {
              hourName = e.hourName
            }
          })
          this.showTime = lunar.year + '-' + lunar.month + '-' + lunar.day + ' (公历)  /  ' + this.dayInfo.GanZhiYear + '年 ' + this.dayInfo.lunarMonthName + ' ' + this.dayInfo.lunarDayName + ' ' + hourName + ' (农历)'
        }
      },
      // 快速切换上个月
      prevNlMonth () {
        var nlmonth = this.nlmonth
        nlmonth -= 1
        this.nlmonth = nlmonth
        if (nlmonth <= 0) { // 如果月份小于1 跳到上一年12月
          this.nlyear = this.nlyear - 1
          this.handleNlYearChange(this.nlyear)
          this.nlmonth = 12
        }
        this.handleNlMonthChange(this.nlmonth)
      },
      // 快速切换下个月
      nextNlMonth () {
        var nlmonth = this.nlmonth
        nlmonth += 1
        this.nlmonth = nlmonth
        if (nlmonth > this.monthArr.length) { // 如果月份大于12 跳到下一年一月
          this.nlyear = this.nlyear + 1
          this.handleNlYearChange(this.nlyear)
          this.nlmonth = 1
        }
        this.handleNlMonthChange(this.nlmonth)
      },
      handleNlYearChange (year) {
        this.nlyear = year
        this.monthArr = this.getMonths(this.nlyear)
        // console.log(this.monthArr)
        var runMonth = this.getRunMonth(this.nlyear)
        if (runMonth === 0 && this.nlmonth === 13) {
          this.nlmonth = 12
        }
        let nlDays = this.getNlDays(this.nlyear, this.nlmonth)
        this.nlDayForArr = nlDays.days
        this.chooseNlDay(this.nlyear, this.nlmonth, this.nlday, false)
        // console.log(this.nlDayForArr)
      },
      // 月份选择处理
      handleNlMonthChange (month) {
        console.log(month)
        this.nlmonth = month
        let nlDays = this.getNlDays(this.nlyear, this.nlmonth)
        this.nlDayForArr = nlDays.days
        if (nlDays.maxDay < this.nlday) {
          this.nlday = nlDays.maxDay
        }
        this.chooseNlDay(this.nlyear, this.nlmonth, this.nlday, false)
      },
      // 快速返回今天
      goToNlDay () {
        var date = new Date()
        var year = date.getFullYear()
        var month = date.getMonth() + 1
        var day = date.getDate()
        var hour = date.getHours()
        this.changeNlDay([year, month, day, hour], false)
      },
      // 以下为通用函数
      // 获取周数据
      getWeek (dateString) {
        var date
        if (dateString === null || typeof dateString === 'undefined') {
          date = new Date()
        } else {
          var dateArray = dateString.split('-')
          date = new Date(dateArray[0], parseInt(dateArray[1] - 1), dateArray[2])
        }
        return '星期' + '日一二三四五六'.charAt(date.getDay())
      },
      // 阴历函数 传入阴历年 阴历月1-13 得到该月份的全部日期
      getNlDays (year, month) {
        // 该算法需要读取7*6的数组 总共42天的数据
        // 先读取31天的数据
        var lunarArr = []
        var solarDate = ''
        for (var i = 1; i <= 31; i++) {
          var lunar = LunarCalendar.lunarToSolar(year, month, i) // 阴历转公历 再公历转阴历获取更加详细的阴历数据
          if (i === 1) {
            solarDate = lunar.year + '-' + (lunar.month <= 3 ? '0' + lunar.month : lunar.month) + '-' + (lunar.day <= 9 ? '0' + lunar.day : lunar.day)
          }
          var solarToLunar = LunarCalendar.solarToLunar(lunar.year, lunar.month, lunar.day)
          solarToLunar.gl_year = lunar.year
          solarToLunar.gl_month = lunar.month
          solarToLunar.gl_day = lunar.day
          lunarArr.push(solarToLunar)
        }
        // 计算左边需要额外加入多少数据
        var dateArray = solarDate.split('-')
        var date = new Date(dateArray[0], parseInt(dateArray[1] - 1), dateArray[2])
        var leftNum = date.getDay()
        for (var j = 0; j < leftNum; j++) {
          if (j === 0) {
            lunar = LunarCalendar.lunarToSolar(year, month, -1)
            solarToLunar = LunarCalendar.solarToLunar(lunar.year, lunar.month, lunar.day + 1)
            solarToLunar.gl_year = lunar.year
            solarToLunar.gl_month = lunar.month
            solarToLunar.gl_day = lunar.day + 1
            lunarArr.unshift(solarToLunar)
            continue
          }
          lunar = LunarCalendar.lunarToSolar(year, month, -j)
          solarToLunar = LunarCalendar.solarToLunar(lunar.year, lunar.month, lunar.day)
          solarToLunar.gl_year = lunar.year
          solarToLunar.gl_month = lunar.month
          solarToLunar.gl_day = lunar.day
          lunarArr.unshift(solarToLunar)
        }
        // 计算右边需要填充多少数据
        var maxLen = 6 * 7
        var rightNum = maxLen - lunarArr.length
        for (var u = 1; u <= rightNum; u++) {
          lunar = LunarCalendar.lunarToSolar(year, month, u + 31)
          solarToLunar = LunarCalendar.solarToLunar(lunar.year, lunar.month, lunar.day)
          solarToLunar.gl_year = lunar.year
          solarToLunar.gl_month = lunar.month
          solarToLunar.gl_day = lunar.day
          lunarArr.push(solarToLunar)
        }
        // 计算当前月总共几天
        var maxDay = 1
        lunarArr.forEach(function (e, i) {
          if (e.lunarMonth === month && e.lunarDay >= maxDay) {
            maxDay = e.lunarDay
          }
        })
        var dayForArr = []
        for (let i in [0, 1, 2, 3, 4, 5]) {
          var start = i * 7
          var end = (i / 1 + 1) * 7
          dayForArr.push(lunarArr.slice(start, end))
        }
        this.leftHight = '430px'
        if (dayForArr[5][0].lunarMonth !== this.nlmonth) {
          this.leftHight = '375px'
        }
        let result = {}
        result.maxDay = maxDay
        result.days = dayForArr
        return result
      },
      // 传入年 获取该年可选择的月份数据
      getMonths (year) {
        var runMonth = this.getRunMonth(year)
        var monthArr = ['正月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月']
        if (runMonth !== 0) {
          var runMonthName = '闰' + monthArr[runMonth - 1]
          monthArr.splice(runMonth, 0, runMonthName)
        }
        return monthArr
      },
      // 传入年 获取改年润几月数据
      getRunMonth (year) {
        var table = [
          0x04bd8, 0x04ae0, 0x0a570, 0x054d5, 0x0d260, 0x0d950, 0x16554, 0x056a0, 0x09ad0,
          0x055d2, 0x04ae0, 0x0a5b6, 0x0a4d0, 0x0d250, 0x1d255, 0x0b540, 0x0d6a0, 0x0ada2,
          0x095b0, 0x14977, 0x04970, 0x0a4b0, 0x0b4b5, 0x06a50, 0x06d40, 0x1ab54, 0x02b60,
          0x09570, 0x052f2, 0x04970, 0x06566, 0x0d4a0, 0x0ea50, 0x06e95, 0x05ad0, 0x02b60,
          0x186e3, 0x092e0, 0x1c8d7, 0x0c950, 0x0d4a0, 0x1d8a6, 0x0b550, 0x056a0, 0x1a5b4,
          0x025d0, 0x092d0, 0x0d2b2, 0x0a950, 0x0b557, 0x06ca0, 0x0b550, 0x15355, 0x04da0,
          0x0a5d0, 0x14573, 0x052d0, 0x0a9a8, 0x0e950, 0x06aa0, 0x0aea6, 0x0ab50, 0x04b60,
          0x0aae4, 0x0a570, 0x05260, 0x0f263, 0x0d950, 0x05b57, 0x056a0, 0x096d0, 0x04dd5,
          0x04ad0, 0x0a4d0, 0x0d4d4, 0x0d250, 0x0d558, 0x0b540, 0x0b5a0, 0x195a6, 0x095b0,
          0x049b0, 0x0a974, 0x0a4b0, 0x0b27a, 0x06a50, 0x06d40, 0x0af46, 0x0ab60, 0x09570,
          0x04af5, 0x04970, 0x064b0, 0x074a3, 0x0ea50, 0x06b58, 0x055c0, 0x0ab60, 0x096d5,
          0x092e0, 0x0c960, 0x0d954, 0x0d4a0, 0x0da50, 0x07552, 0x056a0, 0x0abb7, 0x025d0,
          0x092d0, 0x0cab5, 0x0a950, 0x0b4a0, 0x0baa4, 0x0ad50, 0x055d9, 0x04ba0, 0x0a5b0,
          0x15176, 0x052b0, 0x0a930, 0x07954, 0x06aa0, 0x0ad50, 0x05b52, 0x04b60, 0x0a6e6,
          0x0a4e0, 0x0d260, 0x0ea65, 0x0d530, 0x05aa0, 0x076a3, 0x096d0, 0x04bd7, 0x04ad0,
          0x0a4d0, 0x1d0b6, 0x0d250, 0x0d520, 0x0dd45, 0x0b5a0, 0x056d0, 0x055b2, 0x049b0,
          0x0a577, 0x0a4b0, 0x0aa50, 0x1b255, 0x06d20, 0x0ada0
        ]
        return table[year - 1900] & 0xf
      }
    },
    mounted: function () {
      this.switchType = this.dateType
      if (this.switchType / 1 === 1) {
        this.changeDay(this.dateArr, true)
        return
      }
      this.changeNlDay(this.dateArr, true)
    },
    computed: {
      weekDay: function () {
        return this.getWeek(this.year + '-' + (this.month <= 9 ? '0' + this.month : this.month) + '-' + (this.day <= 9 ? '0' + this.day : this.day))
      }
    },
    watch: {
      dateArr: function (newVal, oldVal) {
        if (this.switchType / 1 === 1) {
          this.changeDay(newVal, true)
          return
        }
        this.changeNlDay(newVal, true)
      },
      dayArr: function (newVal) {
        let arr = newVal
        var dayForArr = []
        for (let i in [0, 1, 2, 3, 4, 5]) {
          var start = i * 7
          var end = (i / 1 + 1) * 7
          dayForArr.push(arr.slice(start, end))
        }
        this.leftHight = '430px'
        if (dayForArr[5][0].month !== this.month) {
          this.leftHight = '375px'
        }
        this.dayForArr = dayForArr
      }
    }
  }
</script>
<style>
* {
  box-sizing: border-box;
}
.kalendar {
  width: 100%;
  height: 100%;
}
.op-calendar-new-box {
  border-right:0;
  overflow: hidden;
  /* height: 422px !important; */
  position:relative;
  z-index:1
}
.op-calendar-new-holiday-box,.op-calendar-new-month-box,.op-calendar-new-year-box {
  float:left
}
.op-calendar-new-month-box,.op-calendar-new-year-box {
  margin-right:7px
}
.op-calendar-new-year-box {
  width:80px
}
.op-calendar-new-month-box {
  width:61px;
  padding:0 22px;
  position:relative;
  z-index:1
}
.op-calendar-new-holiday-box {
  width:95px
}
.op-calendar-new-select-box {
  width: 335px;
  height:40px;
  float:left;
  zoom:1;
  visibility:hidden;
}
.op-calendar-new-backtoday {
  float:right;
  width:56px;
  margin-top: 7px;
  color: #5EAEFF;
}
.op-calendar-new-left {
  padding:10px;
  width: 405px;
  float: left;
  z-index:1;
  left:0
}
.op-calendar-new-right {
  color:#fff;
  text-align:center;
  margin-left:406px;
  /* height: 100% !important; */
  filter:progid:DXImageTransform.Microsoft.gradient(startColorstr=#55aaff,endColorstr=#73b9ff,grandientType=0);
  background:#5caeff;
  background:-webkit-gradient(linear,0 0,0 100%,from(#5af),to(#73b9ff));
  background:-moz-linear-gradient(top,#5af,#73b9ff)
}
.op-calendar-new-table-box {
  float:left
}
.op-calendar-new-month-box .c-dropdown2-btn-icon-border {
  border-color:transparent;
  _border-color:#fff;
  background-color:transparent!important
}
.op-calendar-new .c-dropdown2 .c-dropdown2-btn-icon {
  padding-left:0
}
.op-calendar-new-next-month,.op-calendar-new-prev-month {
  position:absolute;
  top:0;
  display:block;
  width:21px;
  border:1px solid #999;
  border-bottom-color:#d8d8d8;
  background:#fafafa;
  height:24px;
  text-align:center;
  line-height:24px;
  text-decoration:none;
  color:#7a7a7a;
  font-weight:700;
  font-family:Simsun,Simhei,sans-serif;
  z-index:205
}
.op-calendar-new-prev-month {
  left:0;
  border-right-color:#d8d8d8
}
.op-calendar-new-next-month {
  right:0;
  border-left-color:#d8d8d8
}
.op-calendar-new-next-month:hover,.op-calendar-new-prev-month:hover {
  color:#389cff;
  border-color:#389cff;
  border-left:1px solid #389cff
}
.op-calendar-new-month-box .c-dropdown2-btn {
  _text-indent:-2px
}
.op-calendar-new-table {
  width:385px;
  border-collapse:collapse;
  border-spacing:0
}
.op-calendar-new-table td,.op-calendar-new-table th {
  width:55px;
  height:54px;
  border-top:1px solid #c8cacc;
  padding:0
}
.noTopBorder th{
  border-top: none;
}
.op-calendar-new-relative {
  position:relative;
  width:100%;
  zoom:1
}
.op-calendar-new-table th {
  height:33px;
  border-color:#5af;
  font-weight:400;
  font-size:14px
}
.op-calendar-new-table-six td {
  height:45px
}
.op-calendar-new-table td a {
  display:block;
  width: 55px !important;
  border-right:1px solid #fff;
  height:55px !important;
  padding:11px 3px 10px;
  text-align:center;
  text-decoration:none;
  line-height:1;
  white-space:nowrap;
  overflow:hidden
}
.op-calendar-new-table td a * {
  cursor:pointer;
  margin-bottom: 5px;
}
.op-calendar-new-table-six td a {
  padding:6px 3px
}
.op-calendar-new-table td .op-calendar-new-table-border,.op-calendar-new-table td .op-calendar-new-table-selected,.op-calendar-new-table td a:hover {
  padding: 8px 0 7px;
  width: 55px !important;
  border:3px solid #fb0
}
.op-calendar-new-table-six td .op-calendar-new-table-border,.op-calendar-new-table-six td .op-calendar-new-table-selected,.op-calendar-new-table-six td a:hover {
  padding:3px 0
}
.op-calendar-new-daynumber {
  display:block;
  height:22px;
  font-size:18px;
  color:#000
}
.op-calendar-new-table-almanac {
  display:block;
  color:#999;
  font-size:12px
}
.op-calendar-new-table-festival .op-calendar-new-table-almanac,.op-calendar-new-table-weekend .op-calendar-new-daynumber,th.op-calendar-new-table-weekend {
  color:#e02d2d
}
.op-calendar-new-table-other-month .op-calendar-new-daynumber,.op-calendar-new-table-other-month .op-calendar-new-table-almanac {
  color:#bfbfbf
}
.op-calendar-new-table-today .op-calendar-new-daynumber,.op-calendar-new-table-today .op-calendar-new-table-almanac {
  color:#fff
}
.op-calendar-new-table-today {
  background:#fb0
}
.op-calendar-new-table td .op-calendar-new-table-rest,.op-calendar-new-table td .op-calendar-new-table-work {
  background:#fff0f0
}
.op-calendar-new-table td .op-calendar-new-table-work {
  background:#f5f5f5
}
.op-calendar-new-table-holiday-sign {
  position:absolute;
  left:0;
  top:0;
  display:block;
  width:15px;
  height:15px;
  color:#fff;
  background:#f43;
  text-align:left;
  text-indent:1px;
  line-height:14px;
  *line-height:18px;
  overflow:hidden
}
.op-calendar-new-table-work .op-calendar-new-table-holiday-sign {
  background:#969799
}
.op-calendar-new-table-other-month .op-calendar-new-table-holiday-sign {
  filter:alpha(opacity=50);
  opacity:.5
}
.op-calendar-new-right-date {
  height:48px;
  line-height:48px
}
.op-calendar-new-right-day {
  position:relative;
  width:75px;
  height:75px;
  margin:0 auto;
  line-height:76px;
  font-size:52px;
  background:#fb0;
  border-radius:3px;
  box-shadow:1px 2px 5px rgba(0,0,0,.1),-1px 2px 5px rgba(0,0,0,.1);
  margin-bottom: 20px;
}
.op-calendar-new-right-lunar span {
  display:block;
  margin-top: 5px;
}
.op-calendar-new-right-almanac {
  margin:10px auto 0;
  width:110px;
  border-top:2px solid #94c9ff;
  padding-top:10px;
  line-height:18px;
  height:155px
}
.op-calendar-new-right-almanac span {
  display:block;
  width:55px;
  float:left;
  white-space:nowrap;
  overflow:hidden
}
.op-calendar-new-right-almanac i {
  display:block;
  width:30px;
  height:30px;
  margin:0 auto 5px
}
.op-calendar-hover-avoid i,.op-calendar-hover-suit i,.op-calendar-new-right-almanac i {
  font:24px/30px 'Microsoft Yahei';
  text-shadow:2px 2px 1px rgba(0,0,0,.1);
  text-align:center;
  color:#fff
}
.op-calendar-new-right-hover .op-calendar-hover-almanac {
  display:block
}
.op-calendar-hover-almanac {
  display:none;
  position:absolute;
  width:210px;
  right:-231px;
  top:198px;
  background:#fff;
  padding:15px 10px;
  border:1px solid #5fafff;
  color:#333;
  box-shadow:4px 4px rgba(0,0,0,.05);
  z-index:100
}
.op-calendar-almanac-arrow {
  position:absolute;
  top:20px;
  left:-11px;
  font:22px Simsun;
  color:#fff;
  text-shadow:0 -1px rgba(0,0,0,.05);
  z-index:1
}
.op-calendar-hover-avoid,.op-calendar-hover-suit {
  padding-left:40px;
  position:relative;
  display:block;
  min-height:30px;
  height:30px;
  text-align:left
}
.op-calendar-hover-avoid i,.op-calendar-hover-suit i {
  display:block;
  position:absolute;
  top:0;
  left:0;
  width:30px;
  height:30px;
  background:#67b3ff
}
.op-calendar-hover-avoid i {
  background:#ff5040
}
.op-calendar-new-holidaytip {
  display:none;
  position:relative;
  background:#f7f7f7;
  padding:10px 10px 10px 0
}
.op-calendar-new-holidaytip p {
  margin-left:35px
}
.op-calendar-new-holidaytip-icon {
  position:absolute;
  left:0;
  top:10px;
  padding-left:10px;
  width:20px;
  height:20px;
  text-align:center;
  font:20px/20px Simsun;
  color:#61b0ff
}
.op-calendar-new-holidaytip-icon i {
  font:14px/20px Tahoma,Arial;
  _line-height:16px;
  position:absolute;
  width:20px;
  height:20px;
  right:0;
  top:0
}
.op-calendar-new-holidaystyle .op-calendar-new-box,.op-calendar-new-red-bg .op-calendar-new-box {
  border-color:#cb1c18
}
.op-calendar-new-holidaystyle .op-calendar-new-right,.op-calendar-new-red-bg .op-calendar-new-right {
  color:#fff;
  text-align:center;
  margin-left:406px;
  height:368px;
  height:366px;
  filter:progid:DXImageTransform.Microsoft.gradient(startColorstr=#cb1c18,endColorstr=#f44f23,grandientType=0);
  background:#cb1c18;
  background:-webkit-gradient(linear,0 0,0 100%,from(#cb1c18),to(#f44f23));
  background:-moz-linear-gradient(top,#cb1c18,#f44f23)
}
.op-calendar-new-holidaystyle .op-calendar-new-right-almanac,.op-calendar-new-red-bg .op-calendar-new-right-almanac {
  border-top-color:#eb7563
}
.op-calendar-new-holidaystyle .op-calendar-new-table th,.op-calendar-new-red-bg .op-calendar-new-table th {
  border-color:#f55c4e
}
.op-calendar-new-holidaystyle .op-calendar-hover-almanac,.op-calendar-new-red-bg .op-calendar-hover-almanac {
  border-color:#cb1c18
}
.op-bk-polysemy-bold {
  font-weight:700
}
.op-bk-polysemy-other span {
  display:block
}
.op-bk-polysemy-space {
  white-space:nowrap
}
.op-bk-polysemy-oneother .op-bk-polysemy-move,.op-bk-polysemy-oneother span {
  margin-left:0
}
.op-bk-polysemy-focus {
  height:22px;
  height:24px;
  line-height:1.69em;
  margin-bottom:2px;
  overflow:hidden
}
.op-bk-polysemy-focustext {
  float:left;
  display:inline-block;
  height:22px;
  padding:0 8px 0 30px;
  background:url(//www.baidu.com/aladdin/img/bk_polysemy/bk_polyicon.png) 5px 0 no-repeat #3288ff;
  _background:url(//www.baidu.com/aladdin/img/bk_polysemy/bk_polyicon1.gif) 5px center no-repeat #3288ff;
  color:#fff
}
.op-bk-polysemy-focus a {
  display:inline-block;
  height:22px;
  line-height:1.69em;
  float:left;
  border-right:2px solid #fff;
  background:#f5f5f5;
  padding:0 8px;
  text-decoration:none;
  color:#333
}
.op-bk-polysemy-focus a.op-bk-polysemy_focusafirst {
  background:url(//www.baidu.com/aladdin/img/bk_polysemy/bk_polyicon.png) 0 -22px no-repeat #f5f5f5
}
.op-bk-polysemy-focusf {
  height:23px;
  height:25px;
  line-height:1.69em;
  margin-bottom:2px;
  overflow:hidden
}
.op-bk-polysemy-focusleft {
  float:left;
  display:inline-block;
  height:21px;
  border:1px solid #38f
}
.op-bk-polysemy-focusrightf,.op-bk-polysemy-focustextf {
  border-top:1px solid #f0f0f0;
  border-bottom:1px solid #f0f0f0
}
.op-bk-polysemy-focustextf {
  float:left;
  display:inline-block;
  height:21px;
  padding-left:10px
}
.op-bk-polysemy-focustextf span {
  color:#38f
}
.op-bk-polysemy-focusrightf {
  float:left;
  display:inline-block;
  height:21px;
  border-right:1px solid #f0f0f0
}
.op-bk-polysemy-focusrightf span {
  display:inline-block;
  float:left;
  color:#ccc
}
.op-bk-polysemy-focusrightf a {
  display:inline-block;
  height:21px;
  line-height:1.54em;
  text-decoration:underline;
  border:none;
  background:#fff;
  float:left;
  padding:0 8px;
  color:#00c
}
.op-bk-polysemy-focusrightf a.op-bk-polysemy_focusrfirst {
  padding-left:4px;
  background:#fff;
  color:#00c
}
.op-bk-polysemy-album {
  position:relative;
  width:100%;
  display:block
}
.op-bk-polysemy-albumPr {
  position:relative
}
.op-bk-polysemy-albumMore {
  display:block;
  width:100%;
  height:18px;
  line-height:18px;
  background:#525252;
  background:rgba(82,82,82,.6);
  color:#fff;
  position:absolute;
  bottom:0;
  left:0;
  text-align:center;
  filter:alpha(opacity=60)
}
.op-bk-polysemy-albumBorder {
  width:99%;
  height:99%;
  position:absolute;
  border-right:1px solid #bfbfbf;
  border-bottom:1px solid #bfbfbf;
  right:-2px;
  bottom:-2px;
  overflow:hidden;
  z-index:59;
  _right:-3px
}
.op-bk-polysemy-albumBorderSec {
  right:-4px;
  bottom:-4px;
  _right:-5px
}
.opr-recommends-merge-title {
  text-decoration:none
}
.opr-recommends-merge-title:hover {
  text-decoration:underline
}
.opr-recommends-merge-imgtext {
  display:block
}
.opr-recommends-merge-hide {
  display:none
}
.opr-recommends-merge-p {
  position:relative;
  _zoom:1
}
.opr-recommends-merge-d {
  min-height:20px;
  color:#999
}
.opr-recommends-merge-width-text {
  width:70px;
  text-align:center;
  margin:3px auto 0;
  font-size:12px;
  line-height:18px;
  word-break:break-all
}
.opr-recommends-merge-item {
  text-align:center
}
.opr-recommends-merge-mask {
  position:absolute;
  top:0;
  left:0;
  width:100%;
  _background:0 0;
  background:-webkit-radial-gradient(center,closest-side,rgba(255,255,255,0),rgba(0,0,0,.03));
  background:-moz-radial-gradient(center,closest-side,rgba(255,255,255,0),rgba(0,0,0,.03));
  background:-o-radial-gradient(center,closest-side,rgba(255,255,255,0),rgba(0,0,0,.03));
  background:-ms-radial-gradient(center,closest-side,rgba(255,255,255,0),rgba(0,0,0,.03))
}
.opr-recommends-merge-more-btn i {
  cursor:pointer
}
.opr-recommends-merge-layerbtn {
  position:absolute;
  right:0;
  bottom:0;
  width:1.23em;
  height:1.23em;
  background:url(//www.baidu.com/aladdin/img/right_recommends/layericon.png) no-repeat;
  _background-image:url(//www.baidu.com/aladdin/img/right_recommends/layericon.gif)
}
.opr-recommends-merge-layerbtn1,.opr-recommends-merge-layerbtn2 {
  background-position:-48px 0
}
.opr-recommends-merge-layerbtn1,.opr-recommends-merge-layerbtn3 {
  background-color:#999
}
.opr-recommends-merge-layerbtn1:hover,.opr-recommends-merge-layerbtn2,.opr-recommends-merge-layerbtn3:hover,.opr-recommends-merge-layerbtn4 {
  background-color:#38f
}
.opr-recommends-merge-layerbtn3:hover,.opr-recommends-merge-layerbtn4:hover {
  background-position:-24px 0
}
.opr-recommends-merge-layer {
  padding:4px 9px;
  width:350px
}
.opr-recommends-merge-layer table {
  border-collapse:collapse;
  border-padding:0
}
.opr-recommends-merge-layer td {
  font-size:1em;
  line-height:1.67;
  vertical-align:top
}
.opr-recommends-merge-lastspan {
  display:none
}
.opr-recommends-merge-mbGap {
  margin-bottom:28px
}
.container_l .opr-recommends-merge-lastspan {
  display:block
}
.container_l .cr-content-narrow .opr-recommends-merge-lastspan {
  display:none
}
.opr-recommends-merge-dodge-wrap {
  margin-bottom:24px;
  font-size:1.1em
}
.opr-recommends-merge-user-layer {
  width:235px;
  position:absolute;
  border:1px solid #eee;
  border-radius:2px;
  margin-top:10px;
  margin-left:-60px;
  *margin-left:-140px;
  z-index:998;
  background:#fff;
  color:#333;
  font-size:13px;
  text-align:center;
  padding:14px 15px
}
.opr-recommends-merge-user-layer button {
  margin-top:12px;
  font-size:12px
}
.opr-recommends-merge-user-layer img {
  top:2px;
  position:relative
}
.opr-recommends-merge-user-secondBtn {
  margin-left:8px
}
.opr-recommends-merge-user-secondBtn i {
  -ms-transform:rotate(180deg);
  -moz-transform:rotate(180deg);
  -webkit-transform:rotate(180deg);
  -o-transform:rotate(180deg)
}
.opr-recommends-merge-user-layer-tips {
  position:absolute;
  margin-top:5px;
  margin-left:67px;
  *margin-left:-22px;
  border-left:6px solid transparent;
  border-right:6px solid transparent;
  border-bottom:6px solid #eee;
  width:0;
  height:0;
  z-index:999
}
.opr-recommends-merge-content {
  position:relative
}
.opr-recommends-merge-user-layer-tips-fff {
  margin-top:6px;
  border-bottom:6px solid #fff
}
.opr-recommends-merge-user-layer-hide {
  display:none
}
.opr-recommends-merge-user-layer-icon {
  position:relative;
  top:2px;
  width:14px;
  height:14px
}
.opr-recommends-merge-user-layer-con {
  position:absolute;
  width:312px;
  height:140px;
  top:0;
  padding-top:20px;
  z-index:999
}
.opr-toplist-title {
  position:relative
}
.opr-toplist-table .opr-toplist-right {
  text-align:right;
  white-space:nowrap
}
.opr-toplist-info {
  color:#666;
  text-align:right
}
.opr-toplist-info a {
  color:#666
}
.opr-toplist-st {
  margin-bottom:2px
}
.container_s #content_right .opui-advert2-img-big {
  width:259px;
  height:70px
}
.container_l #content_right .opui-advert2-img-big {
  width:351px;
  height:95px
}
.opr-toplist-more {
  position:absolute;
  right:-10px;
  top:0
}
.opr-toplist-more-currentBtn {
  color:#666
}
.opr-toplist-more-chevron {
  padding:10px;
  cursor:pointer
}
.opr-toplist-more-chevron-left {
  padding-right:2px
}
.opr-toplist-more-chevron-right {
  padding-left:2px
}
.opr-toplist-more-btn {
  display:inline-block;
  width:4px;
  height:4px;
  margin:0 3px;
  *margin-top:-8px;
  overflow:hidden;
  background:url(//www.baidu.com/aladdin/tpl/right_toplist/toplist_dot.png) -4px 0 no-repeat
}
.opr-toplist-more-currentBtn {
  background-position:0 0
}
.opr-toplist1-title {
  position:relative;
  margin-bottom:.5px
}
.opr-toplist1-table .opr-toplist1-right {
  text-align:right;
  white-space:nowrap
}
.opr-toplist1-from {
  text-align:right
}
.opr-toplist1-from a {
  text-decoration:none
}
.opr-toplist1-new {
  position:relative;
  top:1px
}
.opr-toplist1-st {
  margin-bottom:2px;
  margin-left:3px
}
.opr-toplist1-update {
  float:right
}
.opr-toplist1-refresh {
  font-size:13px;
  font-weight:400;
  text-decoration:none
}
.opr-toplist1-refresh .opr-toplist1-icon {
  background:url(//www.baidu.com/aladdin/tpl/right_toplist1/refresh.png) 0 0/100% 100% no-repeat;
  margin-left:3px;
  width:16px;
  height:16px
}
.year, .month, .selectYearAndMonth{
  float: left;
}
.year>span, .month>span {
  float: left;
  display: inline-block;
  border: 1px solid #dcdfe6;
  width: 15px;
  height: 28px;
  line-height: 28px;
  font-size: 16px;
  font-weight: 700;
  color: #c0c4cc;
}
.toToday > button {
  padding: 7px 12px;
  float: right;
}
.selectYearAndMonth .el-input__inner {
  padding-left: 5px !important;
}
.changeKalendar {
  width: 100%;
  height: 50px;
  background-color: #fff;
  padding-top: 10px;
}
.glTextColor {
  color: #409EFF !important;
}
.yin .is-active > span{
  background-color: #CB1C18 !important;
  border: 1px solid #CB1C18 !important;
  box-shadow: -1px 0 0 0 #CB1C18 !important;
}
.nlDay {
  width: 100px;
  height: 65px;
  font-size: 30px;
  letter-spacing: 5px;
  line-height: 65px;
}
.addBirthday {
  width: 657px;
}
.addLittleBirthday {
  width: 362px;
}
</style>