<template>
 <div class="date-picker-wrapper">
   <div class="date-picker">
     <div class="calendar-table-wrapper">
       <table class="calendar-table">
         <thead>
           <tr>
             <th @click="prevMonth">
               <i><</i>
             </th>
             <th>
               {{ firstMonth.year }}년 {{ firstMonth.month+1 }}월
               <select class="yearselect" v-model="currentYear" v-if="showYearSelect">
                 <option v-for="(year, index) in RangeOfYear" :value="year" :key="index">{{ year }}</option>
               </select>
             </th>
             <th>
               <i></i>
             </th>
           </tr>
         </thead>
         <tbody>
           <tr>
             <th v-for="(weekDay, dayIndex) in weekNames" :key="dayIndex">
               {{ weekDay }}
             </th>
           </tr>
           <tr v-for="(dateRow, rowIndex) in firstCalendar" :key="rowIndex">
             <slot name="date-slot" v-for="(date, dateIndex) in dateRow">
               <td
                 class="calendar-cell"
                 :class="{ 'calendar-cell-scheduled': isSchedule(date), 'calendar-cell-focused': isFocusedDate(date), 'calendar-cell-selected': isSelctedDate(date) }"
                 @click="dateClick(date)"
                 @mouseover="dateHover(date)"
                 @mouseleave="dateHoverOut()"
               >
                 <div class="calendar-cell-content">
                   <div v-if="isMouseOver(date)" class="mouse-over-msg">{{ mouseOverMsg }}</div>
                   <div v-if="date">
                     {{ date.date }}
                   </div>
                   <div v-else></div>
                 </div>
               </td>
             </slot>
           </tr>
         </tbody>
       </table>
     </div>
     <div class="calendar-table-wrapper">
       <table class="calendar-table">
         <thead>
           <tr>
             <th>
               <i></i>
             </th>
             <th colspan="5">
               {{ secondMonth.year }}년 {{ secondMonth.month+1 }}월
               <select class="yearselect" v-model="currentYear" v-if="showYearSelect">
                 <option v-for="(year, index) in RangeOfYear" :value="year" :key="index">{{ year }}</option>
               </select>
             </th>
             <th @click="nextMonth">
               <i>></i>
             </th>
           </tr>
         </thead>
         <tbody>
           <tr>
             <th v-for="(weekDay, dayIndex) in weekNames" :key="dayIndex">
               {{ weekDay }}
             </th>
           </tr>
           <tr v-for="(dateRow, rowIndex) in secondCalendar" :key="rowIndex">
             <slot name="date-slot" v-for="(date, dateIndex) in dateRow">
               <td
                 class="calendar-cell"
                 :class="{ 'calendar-cell-scheduled': isSchedule(date), 'calendar-cell-focused': isFocusedDate(date), 'calendar-cell-selected': isSelctedDate(date) }"
                 @click="dateClick(date)"
                 @mouseover="dateHover(date)"
                 @mouseleave="dateHoverOut()"
               >
                 <div class="calendar-cell-content">
                   <div v-if="isMouseOver(date)" class="mouse-over-msg">{{ mouseOverMsg }}</div>
                   <div v-if="date">{{date.date}}</div>
                   <div v-else> </div>
                 </div>
               </td>
             </slot>
           </tr>
         </tbody>
       </table>
     </div>
   </div>
   <div class="block-schedule">
     <div v-for="schedule in blockSchedule">
       <div class="schedule-wrapper"
         :class="{ 'hover-class': isFocusedSchedule(schedule.block_id) }"
         @mouseover="blockScheduleHover(schedule.block_id)"
         @mouseleave="blockScheduleHoverOut()"
       >
         <div class="schedule-info-box reason" v-if="tempSchedule.block_id !== schedule.block_id">
           {{ schedule.reason }}
         </div>
         <div class="schedule-info-box reason" v-if="tempSchedule.block_id === schedule.block_id">
           <input class="form-control form-control-sm" v-model="tempSchedule.reason"/>
         </div>
         <div class="schedule-info-box" v-if="tempSchedule.block_id !== schedule.block_id">
           {{ stringToKorean(schedule.start_date) }} ~ {{ stringToKorean(schedule.end_date) }}
         </div>
         <div class="schedule-info-box reason" v-if="tempSchedule.block_id === schedule.block_id">
           <div class="form-control form-control-sm" > {{ tempSchedule.start_date | stringToKorean }} </div>
           <div style="margin: 3px;"> ~ </div>
           <div class="form-control form-control-sm" > {{ tempSchedule.end_date | stringToKorean }} </div>
         </div>
         <div class="schedule-info-btn-box">
           <base-button class="btn"
             @click-event="changeBlock(schedule.block_id)"
             :name="tempSchedule.block_id !== schedule.block_id ? '변경':'확인'"
             :isSelected="tempSchedule.block_id === schedule.block_id"
           />
           <base-button class="btn"
             @click-event="cancelChangeBlock()"
             v-if="tempSchedule.block_id === schedule.block_id"
             name="취소"
             :isSelected="true"
           />
           <base-button class="btn" v-if="tempSchedule.block_id !== schedule.block_id" @click-event="deleteBlock(schedule.block_id)" name="삭제"/>
         </div>
       </div>
     </div>
   </div>
   <div class="block-schedule-add">
     <div>
       <div class="schedule-add-wrapper">
         <div class="form-box-reason">
           <label class="label-box">사유</label>
           <div class="form-box">
             <input class="form-control" v-model="reason" />
           </div>
         </div>
         <div class="form-box-date">
           <label class="label-box">기간</label>
           <div class="form-box">
             <div class="form-control" > {{ start_date | stringToKorean }} </div>
             <div style="margin: 5px;">
               ~
             </div>
             <div class="form-control" > {{ end_date | stringToKorean}} </div>
           </div>
         </div>
         <div class="btn-box">
           <base-button class="btn" @click-event="addBlock()" name="추가하기"/>
         </div>
       </div>
     </div>
   </div>
 </div>

</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator'
import { State, Getter, Action, Mutation, namespace } from 'vuex-class'
import moment from 'moment';
import { dateToString, stringToKorean } from '@/utils/date-helper';
import BaseButton from '@/components/BaseButton.vue';
import BaseModal from '@/components/BaseModal.vue';
import * as api from '@/api';
import deepCopy from '@/utils/deepcopy';
@Component({
 components: {
   BaseButton, BaseModal
 },
 methods: {
   stringToKorean: stringToKorean
 },
 filters: {
   stringToKorean: stringToKorean
 }
})
export default class BaseDatePicker extends Vue {
 @Prop() private room_id!: number;
 private start_date: string|null = null;
 private end_date: string|null = null;
 private reason: string = "";
 private showYearSelect: boolean = false;
 private weekNames: Array<string> = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'];
 private firstMonth: { year: number, month: number } = { year: 2019, month: 8 };
 private secondMonth: { year: number, month: number } = { year: 2019, month: 9 };
 private firstCalendar: Array<Array<any>> = [[]];
 private secondCalendar: Array<Array<any>> = [[]];
 private currentYear:number = 2019;
 private currentMonth:number = 8;
 private currentDate: number = 5;
 private mouseOverDate: { year: number, month: number, date: number }|null = null;
 private mouseOverScheduleId: number|null = null;
 private mouseOverMsg: string = "";
 private blockSchedule: Array<any> = [];
 private tempSchedule: any = { block_id: null, reason: "", start_date: "", end_date: "" };
 async created() {
   this.currentYear = moment().year();
   this.currentMonth = moment().month();
   this.currentDate = moment().date();
   await this.initCalendar(this.currentYear, this.currentMonth);
 }
 async initCalendar(year: number, month: number): Promise<void> {
   this.firstMonth = { year: year, month: month };
   this.secondMonth = { year: year, month: month+1 };
   if (this.secondMonth.month === 12) {
     this.secondMonth.year += 1;
     this.secondMonth.month = 0;
   }
   let firstDayOfMonth = new Date(this.firstMonth.year, this.firstMonth.month, 1);
   let lastDayOfMonth = new Date(this.firstMonth.year, this.firstMonth.month+1, 0);
   this.firstCalendar = [];
   let day = moment(firstDayOfMonth).day();
   let date = 1;
   for (let i = 0; i < 6; i++) {
     let calendarRow = [];
     for (let j = 0; j < 7; j++) {
       let cnt = i*7+j+1;
       if (i === 0 && day < cnt) {
         calendarRow.push({ year: this.firstMonth.year, month: this.firstMonth.month, date: date });
         date++;
       } else if (i !== 0 && date <= moment(lastDayOfMonth).date()) {
         calendarRow.push({ year: this.firstMonth.year, month: this.firstMonth.month, date: date });
         date++;
       } else {
         calendarRow.push(null);
       }
     }
     this.firstCalendar.push(calendarRow);
   }
   let firstDayOfNextMonth = new Date(this.secondMonth.year, this.secondMonth.month, 1);
   let lastDayOfNextMonth = new Date(this.secondMonth.year, this.secondMonth.month+1, 0);
   this.secondCalendar = [];
   day = moment(firstDayOfNextMonth).day();
   date = 1;
   for (let i = 0; i < 6; i++) {
     let calendarRow = [];
     for (let j = 0; j < 7; j++) {
       let cnt = i*7+j+1;
       if (i === 0 && day < cnt) {
         calendarRow.push({ year: this.secondMonth.year, month: this.secondMonth.month, date: date });
         date++;
       } else if (i !== 0 && date <= moment(lastDayOfNextMonth).date()) {
         calendarRow.push({ year: this.secondMonth.year, month: this.secondMonth.month, date: date });
         date++;
       } else {
         calendarRow.push(null);
       }
     }
     this.secondCalendar.push(calendarRow);
   }
   await this.fetchBlockSchedule()
 }
 async fetchBlockSchedule(): Promise<void> {
   let startDate = dateToString(new Date(this.firstMonth.year, this.firstMonth.month, 1));
   let endDate = dateToString(new Date(this.secondMonth.year, this.secondMonth.month + 1, 0));
   let result = await api.room.getRoomSchedule({ roomId: this.room_id }, { query: { startDate, endDate }});
   if (result.success) {
     this.blockSchedule = result.schedules;
   }
 }

 // BOOLEAN
 isSchedule(date: { year: number, month: number, date: number }): boolean {
   if (date === null) {
     return false;
   }
   let ret = false;
   this.blockSchedule.forEach(schedule => {
     let target = parseInt(dateToString(new Date(date.year, date.month, date.date)));
     if (target >= parseInt(schedule.start_date) && target <= parseInt(schedule.end_date)) {
       if (schedule.block_id !== this.tempSchedule.block_id) {
         ret = true;
       }
     }
   })
   return ret;
 }
 isMouseOver(date: { year: number, month: number, date: number }): boolean {
   if (this.mouseOverMsg === "") return false;
   if (date === null || this.mouseOverDate === null) return false;
   if (this.mouseOverDate.year === date.year && this.mouseOverDate.month === date.month && this.mouseOverDate.date === date.date) return true;
   return false;
 }
 isFocusedDate(date: { year: number, month: number, date: number }): boolean {
   if (date === null) return false;
   let schedule = this.blockSchedule.find(schedule => {
     let target = parseInt(dateToString(new Date(date.year, date.month, date.date)));
     return (target >= parseInt(schedule.start_date) && target <= parseInt(schedule.end_date));
   })
   if (!schedule) return false;
   return this.isFocusedSchedule(schedule.block_id);
 }
 isFocusedSchedule(blockId: number): boolean {
   return blockId === this.mouseOverScheduleId;
 }
 isSelctedDate(date: { year: number, month: number, date: number }): boolean {
   if (date === null) return false;
   let target = parseInt(dateToString(new Date(date.year, date.month, date.date)));
   if (this.tempSchedule.block_id) {
     return (target >= parseInt(this.tempSchedule.start_date) && target <= parseInt(this.tempSchedule.end_date));
   }
   return (target >= parseInt(this.start_date) && target <= parseInt(this.end_date));
 }
 async isScheduleIn(start: string|null, end: string|null, clicked: string, exclude_block_id?: number): boolean {
   let result = null;
   if (parseInt(clicked) >= parseInt(start) && parseInt(clicked) <= parseInt(end)) {
     return true;
   } else if (parseInt(clicked) < parseInt(start)){
     result = await api.room.getRoomIsAvailable({ roomId: this.room_id }, { query: { fromDate: clicked, toDate: start }});
   } else {
     result = await api.room.getRoomIsAvailable({ roomId: this.room_id }, { query: { fromDate: end, toDate: clicked }});
   }
   if (result.success) {
     if (exclude_block_id) {
       let isBlock = result.blocks.find(block => {
         return (exclude_block_id === block.block_id);
       });
       if (isBlock && result.blocks.length === 1) {
         return true;
       }
     }
     return result.available;
   } else {
     return false;
   }
 }

 // HOVER event
 dateHover(date: {year: number, month: number, date: number}): void {
   this.mouseOverDate = date;
   this.mouseOverMsg = "";
   if (this.mouseOverDate === null) return;
   let schedule = this.blockSchedule.find(schedule => {
     let target = parseInt(dateToString(new Date(date.year, date.month, date.date)));
     return (target >= parseInt(schedule.start_date) && target <= parseInt(schedule.end_date));
   })
   if (schedule) {
     this.mouseOverScheduleId = schedule.block_id;
     this.mouseOverMsg = schedule.reason;
     return;
   }
   return;
 }
 dateHoverOut() {
   this.mouseOverScheduleId = null;
   this.mouseOverDate = null;
   this.mouseOverMsg = "";
 }
 blockScheduleHover(blockId: number): void {
   this.mouseOverScheduleId = blockId;
 }
 blockScheduleHoverOut(): void {
   this.mouseOverScheduleId = null;
 }

 // CLICK event
 async dateClick(date: {year: number, month: number, date: number}): Promise<void> {
   if (date === null) return;
   if (this.isSchedule(date)) {
     alert("선택할 수 없습니다!")
     return;
   }
   let clicked = dateToString(new Date(date.year, date.month, date.date));
   if (this.tempSchedule.block_id) {
     let result = await this.isScheduleIn(this.tempSchedule.start_date, this.tempSchedule.end_date, clicked, this.tempSchedule.block_id);
     if (result) {
       let dates = this.getClickDate(this.tempSchedule.start_date, this.tempSchedule.end_date, clicked);
       this.tempSchedule.start_date = dates.start;
       this.tempSchedule.end_date =  dates.end;
     } else {
       alert("선택할 수 없습니다!")
       return;
     }
   } else {
     let result = await this.isScheduleIn(this.start_date, this.end_date, clicked);
     if (result) {
       let dates = this.getClickDate(this.start_date, this.end_date, clicked);
       this.start_date = dates.start;
       this.end_date = dates.end;
     } else {
       alert("선택할 수 없습니다!")
       return;
     }
   }
 }
 getClickDate(start: string|null, end: string|null, clicked: string): { start: string , end: string } {
   if (start) {
     if (start === end) {
       if (parseInt(clicked) >= parseInt(start)) {
         end = clicked;
       } else {
         end = start;
         start = clicked;
       }
     } else {
       if (parseInt(clicked) >= parseInt(start) &&
       parseInt(clicked) <= parseInt(end)) {
         start = null;
         end = null;
       } else if (parseInt(clicked) >= parseInt(end)) {
         end = clicked;
       } else {
         start = clicked;
       }
     }
   } else {
     start = clicked;
     end = start;
   }
   return { start, end };
 }
 async addBlock(): Promise<void> {
   let result = await api.block.postBlock({ body: {
     start_date: this.start_date,
     end_date: this.end_date,
     start_time: 0,
     end_time: 24,
     reason: this.reason,
     roomId: this.room_id
   }});
   if (result.success) {
     await this.fetchBlockSchedule();
     this.reason = "";
     this.start_date = null;
     this.end_date = null;
   }
   alert(result.msg);
 }
 async changeBlock(blockId: number): Promise<void> {
   if (this.tempSchedule.block_id === blockId) { // api 요청
     let result = await api.block.putBlock({ blockId: this.tempSchedule.block_id }, { body: {
       start_date: this.tempSchedule.start_date,
       end_date: this.tempSchedule.end_date,
       start_time: 0,
       end_time: 24,
       reason: this.tempSchedule.reason,
       roomId: this.room_id
     }});
     if (result.success) {
       await this.fetchBlockSchedule();
       this.mouseOverScheduleId = null;
       this.tempSchedule = { block_id: null, reason: "", start_date: "", end_date: "" };
     }
     alert(result.msg);
   } else {
     this.mouseOverScheduleId = blockId;
     this.tempSchedule = deepCopy(this.blockSchedule.find(block => {
       return blockId === block.block_id
     }));
     this.start_date = null;
     this.end_date = null;
   }
 }
 cancelChangeBlock(): void {
   this.mouseOverScheduleId = null;
   this.tempSchedule = { block_id: null, reason: "", start_date: "", end_date: "" };
 }
 async deleteBlock(blockId: number): Promise<void> {
   let result = await api.block.delBlock({ blockId: blockId }, {});
   if (result.success) {
     await this.fetchBlockSchedule();
   }
   alert(result.msg)
 }
 nextMonth(): void {
   if (this.firstMonth.month === 11) {
     this.initCalendar(this.firstMonth.year + 1, 0);
   } else {
     this.initCalendar(this.firstMonth.year, this.firstMonth.month + 1);
   }
 }
 prevMonth(): void {
   if (this.firstMonth.month === 0) {
     this.initCalendar(this.firstMonth.year - 1, 11);
   } else {
     this.initCalendar(this.firstMonth.year, this.firstMonth.month - 1);
   }
 }
 async setYear(year: number): Promise<void> {
   this.currentYear = year;
   await this.initCalendar(this.currentYear, this.currentMonth);
 }

}
</script>

<style lang="scss" scoped>
.date-picker-wrapper {
 position: relative;
 height: 310px;
}
.date-picker {
 display: flex;
}
.calendar-table-wrapper {
 flex: 1;
 border: 0.5px solid #eee;
 border-radius: 15px;
 width: 100%;
 margin: 5px;
 padding: 10px;
}
.calendar-table {
 width: 100%;
}
.calendar-table thead {
 text-align: center;
}
.calendar-table thead tr{
 display: flex;
}
.calendar-table thead tr th{
 text-align: center;
 flex: 1;
}
.calendar-table tbody tr {
 display: flex;
}
.calendar-table tbody tr th{
 text-align: center;
 flex: 1;
 font-size: 0.8em;
}
.calendar-table tbody tr td {
 text-align: center;
 flex: 1;
 font-size: 0.9em;
}
.calendar-cell {
 background-color: #fff;
}
.calendar-cell-content {
 position: relative;
 cursor: Pointer;
}
.calendar-cell-scheduled {
 background-color: #ccc;
}

.calendar-cell-focused {
 background-color: #006600;
 color: white;
}
.calendar-cell-selected {
 background-color: #cccc00;
 color: black;
}
.schedule-wrapper {
 padding: 3px;
 margin-bottom: 5px;
 border: solid 1px #ddd;
 border-radius: 10px;
 display: flex;
}
.schedule-info-box {
 padding-top: 7px;
 padding-bottom: 3px;
 padding-left: 15px;
 padding-right: 15px;
 flex: 1;
 display: flex;
}
.btn-wrapper {
 display: flex;
 margin: auto 0;
}
.btn {
 text-align: right;
}
.mouse-over-msg {
 position: absolute;
 padding: 4px;
 //min-width: 70px;
 width: 200px;
 top: -50%;
 left: 50%;
 transform: translate(-50%, -50%);
 border: 0.5px solid #006600;
 border-radius: 10px;
 z-index: 500;
 background-color: white;
 color: black;
}
.hover-class {
 border: #006600 1px solid;
}
.block-schedule {
 margin-top: 5px;
 overflow: scroll;
 max-height: 220px;
}
.block-schedule-add {
 position: absolute;
 bottom: -160px;
 width: 100%
}
.schedule-add-wrapper {
 display: flex;
}
.form-box {
 flex: 5;
 display: flex;
}
.form-box-reason {
 flex: 2;
 display: flex;
}
.form-box-date {
 flex: 3;
 display: flex;
}
.form-control {
 flex: 1;
}
.btn-box {
 flex: 1;
 text-align: right;
}
.label-box {
 flex: 1;
 margin-right: 10px;
 padding-top: 8px;
 text-align: right;
}

</style>
