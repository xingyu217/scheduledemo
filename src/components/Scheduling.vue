<template>
  <div class="scheduling" tabindex="0" v-if="scheduleData" @keydown.17="ctrlKeyDown" @keyup.17="ctrlKeyup">
    <div class="title-container">
         <span>排班详情-{{}}</span>
    </div>
    <div class="header-input" v-if="editing">
        <div class="item">
            <div class="field-group">
                <label>上班时间</label>
                <el-time-select
                    v-model="startTime"
                    :picker-options="{
                        start: '09:00',
                        step: '00:30',
                        end: '18:00'
                    }"
                    placeholder="选择时间">
                </el-time-select>
            </div>
            <div class="field-group">
                <label>下班时间</label>
                <el-time-select
                    v-model="endTime"
                    :picker-options="{
                        start: '09:00',
                        step: '00:30',
                        end: '18:00'
                    }"
                    placeholder="选择时间">
                </el-time-select>
            </div>
        </div>
        <div class="item">
            <div class="field-group">
                <label>时间间隔</label>
                <el-select v-model="duration" placeholder="请选择">
                    <el-option
                    v-for="i in 3"
                    :key="i"
                    :label="i*10"
                    :value="i/6">
                    </el-option>
                </el-select>
            </div>
            <div class="field-group">
                <el-button type="primary">计算</el-button>
                <el-button type="primary">全清</el-button>
            </div>
        </div>
    </div>
    <div class="header-control" v-if="!editing">
        <el-radio-group v-model="week" size="medium">
            <el-radio-button label="本周" ></el-radio-button>
            <el-radio-button label="下周"></el-radio-button>
        </el-radio-group>
    </div>
    <table class="schedule-table" border="1" @mousedown="tableMouseDown" @mouseup="tableMouseUp">
      <tr>
        <th
          v-for="(_, i) in scheduleData.header"
          :key="i"
          :colspan="_.colSpan ? _.colSpan : 1"
        >
          <div class="content">
            <span class="title">
              {{ _.title }}
            </span>
            <span class="subTitle" v-if="_.subTitle">
              {{ _.subTitle }}
            </span>
          </div>
        </th>
      </tr>
      <tr v-for="(_, i) in scheduleData.ItemCount" :key="i" :class="{morning:i<scheduleData.Morning,afternoon:(i>=scheduleData.Morning&&i<scheduleData.Morning+scheduleData.Afternoon),night:(i>=scheduleData.Morning+scheduleData.Afternoon&&i<scheduleData.Morning+scheduleData.Afternoon+scheduleData.Night)}">
        <td
          class="time-group morning"
          :rowspan="scheduleData.Morning"
          v-if="i == 0"
        >
          <img src="@/assets/images/morning.png" />
          <span>上午</span>
        </td>
        <td
          class="time-group afternoon"
          :rowspan="scheduleData.Afternoon"
          v-if="i == scheduleData.Morning"
        >
          <img src="@/assets/images/afternoon.png" />
          <span>下午</span>
        </td>
        <td
          class="time-group night"
          :rowspan="scheduleData.Night"
          v-if="i == scheduleData.Morning + scheduleData.Afternoon"
        >
          <img src="@/assets/images/night.png" />
          <span>晚上</span>
        </td>
        <td class="time-title">
            {{  scheduleData.TimeFormat(i) }}
          </td>
          <td
            :class="{time:true,selected:scheduleData.data[_d][i].selected}"
            :key="'time' + di"
            v-for="(_d, di) in scheduleData.dataKeys"
            @mouseover="tdMouseOver($event,_d,i)"
            @click="tdClick($event,_d,i)"
          ></td>
      </tr>
    </table>
  </div>
</template>
<script lang="ts">
import Vue from "vue";
import { getWeekDay } from "@/utils/dateHelper";
export default Vue.extend({
  props:{
      editing:{
          type:Boolean,
          required:false,
          default:true,
      }
  },
  data(): IPageData {
    return {
      scheduleData: undefined,
      startTime:"9:00",
      endTime:"18:00",
      duration:30,
      ctrlHold:false,
      downHold:false,
      week:"本周"
    };
  },
  methods: {
      tableMouseDown(event:any){
        console.log("down");
          if(!this.downHold){
              this.downHold=true;
          }
      },
      tableMouseUp(event:any){
        console.log("up");
          this.downHold=false;
      },
      ctrlKeyDown(event:any){
        if(!this.ctrlHold){
          console.log("altdown");
          this.ctrlHold=true;
        }
      },
      ctrlKeyup(event:any){
        console.log("altup");
          this.ctrlHold=false;
      },
      tdMouseOver(event:any, key:string,i:number){
          if(this.downHold){
            console.log("over");
              if(!this.ctrlHold){
                  this.scheduleData!.data[key][i].selected=true;
              }else{
                  this.scheduleData!.data[key][i].selected=false;
              }
              this.refreshState();
          }
      },
      tdClick(event:any, key:string,i:number){
        console.log("clicked");
        this.scheduleData!.data[key][i].selected=!this.scheduleData!.data[key][i].selected;
        console.log(this.scheduleData!.data[key][i]);
        this.$forceUpdate();
      },
      refreshState(){
        this.$forceUpdate();
      }
  },
  computed: {},
  created() {
    console.log("load");
    this.scheduleData = new ScheduleVM();
    console.log(`morning: ${this.scheduleData.Morning}`);
    console.log(`afternoon: ${this.scheduleData.Afternoon}`);
    console.log(`night: ${this.scheduleData.Night}`);
  },
});

interface IPageData {
  scheduleData?: ScheduleVM;
  startTime:string;
  endTime:string;
  duration:number;
  ctrlHold:boolean;
  downHold:boolean;
  week:string;
}

interface ISchedule {
  startTime: number;
  endTime: number;
  duration: number;
  data: { [time: string]: IItem[] };
  week: number;
}
enum ItemType {
  header = "header",
  timeTitle = "timeTitle",
  timeGroup = "timeGroup",
  time = "time",
}
interface IItem {
  title?: string;
  subTitle?: string;
  icon?: string;
  selected?: boolean;
  type?: ItemType;
  colSpan?: number;
}

class ScheduleVM implements ISchedule {
  startTime: number = 9.0;
  endTime: number = 18.0;
  duration: number = 0.5;
  data: { [time: string]: IItem[] } = {};
  week: number = 0;
  morningTime: number = 13.0;
  afternoonTime: number = 18.0;
  nightTime: number = 24.0;
  header: IItem[] = [];
  scheduledData: { [d: string]: { start: number; end: number }[] } = {};
  constructor(data?: ISchedule) {
    let d = new Date();
    d.setDate(d.getDate() + 7 * this.week);
    const weekDates = getWeekDay(d.toString());
    let i = 0;
    this.header = [
      {
        title: "时间",
        colSpan: 2,
      },
      {
        title: "周日",
        subTitle: weekDates[i++],
      },
      {
        title: "周一",
        subTitle: weekDates[i++],
      },
      {
        title: "周二",
        subTitle: weekDates[i++],
      },
      {
        title: "周三",
        subTitle: weekDates[i++],
      },
      {
        title: "周四",
        subTitle: weekDates[i++],
      },
      {
        title: "周五",
        subTitle: weekDates[i++],
      },
      {
        title: "周六",
        subTitle: weekDates[i++],
      },
    ];
    if (data) {
      for (const property in data) {
        if (data.hasOwnProperty(property)) {
          (this as any)[property] = (data as any)[property];
        }
      }
    } else {
      this.updateData(true);
    }
  }
  private updateData(init: boolean = false) {
    for (let i = 1; i <= 7; i++) {
      this.data[this.header[i].subTitle!] = new Array(this.ItemCount)
        .fill(null)
        .map<IItem>((_, ni) => {
          if (init) {
            return { seleted: false, type: ItemType.time };
          } else {
            return {
              seleted: this.isSelected(ni, i),
              type: ItemType.time,
            };
          }
        });
    }
  }
  TimeFormat(index: number): string {
    const t:number=this.startTime+this.duration*index;
    const tr=t.toString().split('.');
    const h:number=+tr[0];
    const m:number=t-h;
    const ms=+((m*60).toFixed(0));
    return `${h<10?'0'+h:h}:${ms<10?'0'+ms:ms}`;
  }
  get Morning(): number {
    return +((this.morningTime - this.startTime) / this.duration).toFixed(0);
  }
  get Afternoon(): number {
    return +((this.afternoonTime - this.morningTime) / this.duration).toFixed(0);
  }
  get Night(): number {
    const n=+((this.endTime - this.afternoonTime) / this.duration).toFixed(0);
    return n==0?1:n;
  }
  get dataKeys(): string[] {
    return Object.keys(this.data);
  }

  set Duration(v: number) {
    const d: number = (v / 60).toFixed(3) as any;
    this.duration = d;
    this.updateData();
  }

  get ItemCount():number{
      const i:number=((this.endTime-this.startTime)/this.duration).toFixed(0) as any; 
      return +i+1;
  }

  updateScheduledData() {
    const keys = Object.keys(this.data);
    let sStart: number | undefined = undefined;
    let sEnd: number | undefined = undefined;
    keys.forEach((k, index) => {
      this.scheduledData[k] = [];
      for (let i = 0; i < this.ItemCount; i++) {
        if (this.data[k][i].selected) {
          if (!sStart) {
            sStart = this.startTime + this.duration * i;
          }
          sEnd = this.startTime + this.duration * i;
        } else if (sEnd) {
          this.scheduledData[k].push({
            start: sStart!,
            end: sEnd!,
          });
          sStart = undefined;
          sEnd = undefined;
        }
      }
      if (sEnd) {
        this.scheduledData[k].push({
          start: sStart!,
          end: sEnd!,
        });
        sStart = undefined;
        sEnd = undefined;
      }
    });
  }
  isSelected(index: number, day: number): boolean {
    const t:number=this.startTime + this.duration * index;
    let selected: boolean = false;
    this.scheduledData[this.header[day].subTitle!]?.forEach((_, i) => {
      if (t >= _.start && t <= _.end) {
        selected = true;
        return;
      }
    });
    return selected;
  }
}
</script>
<style lang="scss" scoped>
.scheduling{
    width: 1642px;
    background: #FFFFFF;
    box-shadow: 0px 0px 21px 0px #EDF1F7;
    border-radius: 12px;
    &>.schedule-table{
        width: 1642px;
        height: 1321px;
        border:1px solid #DDE2EF;
        user-select: none;
        .time{
          &.selected{
            background-color: red;
          }
        }
    }
}
</style>