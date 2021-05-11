<template>
  <div tabindex="-1" ref="calendarContainer" @keyup.46="deletePress">
    <button @click="btnClick" value="test">test</button>
    <FullCalendar tabindex="1" ref="fullCalendar" :options="calendarOptions" @keyup.46="deletePress"/>
  </div>
  
</template>
<script lang="ts">
import Vue from 'vue'
import FullCalendar, { diffDates } from '@fullcalendar/vue';
import dayGridPlugin from '@fullcalendar/daygrid';
import interactionPlugin from '@fullcalendar/interaction';
import timeGridPlugin from '@fullcalendar/timegrid';
import bootstrapPlugin from "@fullcalendar/bootstrap";
import listPlugin from "@fullcalendar/list";
import momentPlugin from '@fullcalendar/moment';
import momentTimezonePlugin from '@fullcalendar/moment-timezone'
import zhLocale from "@fullcalendar/core/locales/zh-cn";

export default Vue.extend({
  components:{
    FullCalendar
  },
  data(): IPageData{
    const self:any=this;
    return {
      calendarEvents: [
        {
            start: '2021-05-05T10:00:00',
            end: '2021-05-06T16:00:00',
            display: 'background',
            className:['bg-success'],
            
          }
      ],
      calendarOptions:{//https://fullcalendar.io/docs
      locale:zhLocale,
        headerToolbar:{//https://fullcalendar.io/docs/headerToolbar
          start:'',
          center:'',
          end:'prev,next'
        },
        plugins:[
            dayGridPlugin,
            interactionPlugin,
            timeGridPlugin,
            bootstrapPlugin,
            listPlugin,
            momentTimezonePlugin
          ],
        timeZone: 'Asia/Shanghai',
        initialView: 'timeGridWeek',
        allDaySlot:false,
        nowIndicator:false,
        events: [
        {
            id:1,
            start: '2021-05-04T10:00:00',
            end: '2021-05-04T13:00:00',
            display: 'auto',
            className:['bg-success'],
            editable:true
          },
          {
            id:2,
            start: '2021-05-04T14:00:00',
            end: '2021-05-04T15:00:00',
            display: 'auto',
            className:['bg-success'],
            editable:true
          },
           {
            id:3,
            start: '2021-05-06T10:00:00',
            end: '2021-05-06T15:00:00',
            display: 'auto',
            className:['bg-success'],
            editable:true
          }
      ],
      // titleFormat:{
      //   hour12:true,
      //   hour:'2-digit',
      //   minute:'2-digit',
      // }, //https://fullcalendar.io/docs/date-formatting
      dayHeaderFormat:{
        month:'2-digit',
        weekday:"long",
        day:"2-digit",
        hour12:false
      },
      eventTimeFormat:{
        hour:'2-digit',
        minute:'2-digit',
        hour12:false,
      },
      eventOverlap:false,
        editable: true,
        droppable: true,
        eventResizableFromStart: true,
        weekends: true,
        selectable: true,
        selectMirror: true,
        dayMaxEvents: true,
        dateClick:self.dateClick,
        eventClick: self.editEvent,

        select: self.select
      },
      selectedEle:undefined,
      eventIndex:1,
      startHour:9,
      startMinute:0,
      endHour:18,
      endMinute:0
    }
  },
  created(){
    this.eventIndex=this.calendarOptions.events.length+1;
  },
  methods:{
    dateClick(e:any){
      console.log("======");
      this.$refs.calendarContainer.focus();
    },
    select(e:any){
      console.log(e);
      //   this.calendarEvents.push({
      //     id: this.calendarEvents.length+1,
      //     title: 'test vent',
      //     start: e.start,
      //     end: e.end,
      //     className: 'bg-warning text-white',
      // });
      // const dayDiff=e.end.getDate()-e.start.getDate();
      // let events=this.calendarOptions.events;
      
      //   for(let i=0;i<dayDiff+1;i++){
      //   events=combineArray(events,{
      //     start:i==0?e.start:new Date(new Date((new Date(e.start.getTime())).setHours(0,0)).setDate(e.start.getDate()+i)),
      //     end:i==dayDiff?e.end:new Date(new Date((new Date(e.start.getTime())).setHours(23,59)).setDate(e.start.getDate()+i))
      //     });
      // }
      let data=this.calendarOptions.events;
      data.push({start:e.start,end:e.end});
      
      this.calendarOptions.events=combineArray2(data.sort(compare),9,0,18,0);
      //this.calendarOptions.events= events.sort(compare);
      // this.calendarOptions.events.push({
      //   id:this.eventIndex++,
      //       start: e.start,
      //      end: e.end,
      //       display: 'auto',
      //       className:['bg-success'],
      //       editable:true
      //     });

      this.$forceUpdate();
    },
    t(){
      
    },
    editEvent(e:any){
      console.log(e);
      console.log(e.el.style.backgroundColor);
      if(this.selectedEle){
        this.selectedEle.el.style.backgroundColor="";
      }
      e.el.style.backgroundColor="green"
      this.selectedEle=e;
      this.$refs.calendarContainer.focus();
    },
    deletePress(e:any){
      console.log(e);
      console.log("delete");
      if(this.selectedEle){
        console.log(this.selectedEle.event.id);
        const i=this.calendarOptions.events.findIndex(e=>e.id==this.selectedEle.event.id);
        if(i>=0){
          this.calendarOptions.events.splice(i,1);
        }
      }
      
    },
    btnClick(){
      console.log(this.$refs.fullCalendar);
      let calendarApi = this.$refs.fullCalendar.getApi();
      console.log(calendarApi.getEvents());
    }
  }
})

interface IPageData {
  calendarOptions:any,
  calendarEvents:any[],
  selectedEle:any;
  eventIndex:number;
  startHour:number;
  startMinute:number;
  endHour:number;
  endMinute:number;
}
function compare(a,b){
  const ad=new Date(a.start);
  const ed=new Date(b.start);
  if(ad>ed){
    return 1;
  }
  if(ed>ad){
    return -1;
  }
  if(ad==ed){
    const aed=new Date(a.end);
    const bed=new Date(b.end);
    if(aed>bed){
      return 1;
    }
    if(bed>aed){
      return -1
    }
    return 0;
  }
  return 0;
}
function combineArray(s:any[],data,startHour:number=9,startMinute:number=0,endHour:number=18,endMinute:number=0){
  let match:boolean=false;

  const sd=new Date(data.start);
  const sdTimes=sd.getTime();
  const ed=new Date(data.end);
  const edTimes=ed.getTime();
  const startDate=new Date(new Date(sdTimes).setHours(startHour,startMinute,0,0));
  const endDate=new Date(new Date(edTimes).setHours(endHour,endMinute,0,0));
  let newData=[...s];
  let removeIndex:number[]=[];
  for(let j=0;j<s.length;j++){
    const esd=new Date(s[j].start);
    const eed=new Date(s[j].end);
    if(sd.getDate()==esd.getDate()&&!(ed<esd||sd>eed)){
       removeIndex.push(j);
        match=true;
      }else if(match){
        match=false;
        const k=removeIndex[removeIndex.length-1];
        newData.push({
          start:new Date(new Date(sdTimes).setHours(Math.max(startHour,Math.min(new Date(s[k].start).getHours(),sd.getHours())),new Date(s[k].start).getHours()<sd.getHours()?new Date(s[k].start).getMinutes():sd.getHours()<=startHour?startMinute:sd.getMinutes())),
          end:new Date(new Date(edTimes).setHours(Math.min(endHour,Math.max(new Date(s[k].end).getHours(),ed.getHours())),new Date(s[k].end).getHours()>ed.getHours()?new Date(s[k].end).getMinutes():ed.getHours()>=endHour?endMinute:ed.getMinutes())),
        })
      break;
      }
    if(j==s.length-1){
      if(match){
        match=false;
        const k=removeIndex[removeIndex.length-1];
        //removeIndex.push(k);
        newData.push({
          start:new Date(new Date(sdTimes).setHours(Math.max(startHour,Math.min(new Date(s[k].start).getHours(),sd.getHours())),new Date(s[k].start).getHours()>sd.getHours()?new Date(s[k].start).getMinutes():sd.getHours()<=startHour?startMinute:sd.getMinutes())),
          end:new Date(new Date(edTimes).setHours(Math.min(endHour,Math.max(new Date(s[k].end).getHours(),ed.getHours())),new Date(s[k].end).getHours()>ed.getHours()?new Date(s[k].end).getMinutes():ed.getHours()>=endHour?endMinute:ed.getMinutes())),
        })
      }
       else{
         if(!(sd>endDate||ed<startDate)){
           newData.push({
           start:sd>startDate?sd:startDate,
           end:ed<endDate?ed:endDate,
         });
         }
       } 
      }
  }
  for(let ri=removeIndex.length-1;ri>=0;ri--){
    newData.splice(removeIndex[ri],1);
  }
  
  return newData;
}

function combineArray2(s:any[],startHour:number=9,startMinute:number=0,endHour:number=18,endMinute:number=0){
  let newData=s.length==1?[...s]:[];
  for(let i=1;i<s.length;i++){
    const preStart=new Date(s[i-1].start);
    const preEnd=new Date(s[i-1].end);
    const nextStart=new Date(s[i].start);
    const nextEnd=new Date(s[i].end);

    if(preEnd>=nextStart){
      if(i==s.length-1){
        newData.push({start:preStart,end:preEnd>nextEnd?preEnd:nextEnd});
      }else{
        for(let j=i+1;j<s.length;j++){
          const nextStart2=new Date(s[j].start);
          const nextEnd2=new Date(s[j].end);
          if(preEnd<nextStart2){
            newData.push({
              start:preStart,
              end:new Date(s[j-1].end)>preEnd?new Date(s[j-1].end):preEnd
            });
            
            if(j==s.length-1&&newData[newData.length-1].end<nextEnd2){
              newData.push({start:nextStart2,end:nextEnd2});
            }
            i=j;
            break;
          }else if(j==s.length-1){
            newData.push({start:preStart,end:preEnd>nextEnd2?preEnd:nextEnd2});
            i=j;
          }
        }
      }
      
    }else{
      newData.push({start:preStart,end:preEnd});
      if(i==s.length-1){
        if(newData[newData.length-1].end<nextEnd){
         newData.push({start:nextStart,end:nextEnd});
        }
      }
      
    }
  }
  const currentDate=new Date();
  let splitDate=[];
  
  for(let k=0;k<newData.length;k++){
    const startDay=newData[k].start.getDate();
    const dayDiff=newData[k].end.getDate()-startDay;
    for(let p=0;p<=dayDiff;p++){
      const start=new Date(currentDate.getFullYear(),newData[k].start.getMonth(), startDay+p, p>0?startHour:Math.max(newData[k].start.getHours(),startHour),p>0?startMinute:getMinutes(newData[k].start,startHour,startMinute));
      const end=new Date(currentDate.getFullYear(),start.getMonth(),startDay+p,p<dayDiff?endHour:Math.min(newData[k].end.getHours(),endHour),p<dayDiff?endMinute:getMinutes(newData[k].end,endHour,endMinute,false));
      if(!(start.getHours()>endHour||end.getHours()<startHour)){
          splitDate.push(
          {
            start,
            end
          }
        )
      }
      
    }
    
  }
  return splitDate;
}
function getMinutes(d1:Date,hour:number,minutes:number,max:boolean=true){
  const d1Hours=d1.getHours();
  const d1Minutes=d1.getMinutes();
    if((max?(d1Hours>hour):(d1Hours<hour))){
      return d1Minutes;
    }else if(d1Hours==hour){
      return max?Math.max(d1Minutes,minutes):Math.min(d1Minutes,minutes);
    }else{
      return minutes;
    }
}
</script>
<style lang="scss">
*{
  outline:none;
}
.bg-warning{
  background-color: #8f2715;
}
.bg-success{
  background-color: #e01a00;
}
</style>