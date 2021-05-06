<template>
  <div tabindex="-1" ref="calendarContainer" @keyup.46="deletePress">
    <button @click="btnClick" value="test">test</button>
    <FullCalendar tabindex="1" ref="fullCalendar" :options="calendarOptions" @keyup.46="deletePress"/>
  </div>
  
</template>
<script lang="ts">
import Vue from 'vue'
import FullCalendar from '@fullcalendar/vue';
import dayGridPlugin from '@fullcalendar/daygrid';
import interactionPlugin from '@fullcalendar/interaction';
import timeGridPlugin from '@fullcalendar/timegrid';
import bootstrapPlugin from "@fullcalendar/bootstrap";
import listPlugin from "@fullcalendar/list";
import momentPlugin from '@fullcalendar/moment';
import momentTimezonePlugin from '@fullcalendar/moment-timezone'
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
            start: '2021-05-05T10:00:00',
            end: '2021-05-06T16:00:00',
            display: 'auto',
            className:['bg-success'],
            editable:true
          },
          {
            id:2,
            start: '2021-05-04T10:00:00',
            end: '2021-05-04T16:00:00',
            display: 'auto',
            className:['bg-success'],
            editable:true
          }
      ],
      titleFormat:{
        hour12:true,
        hour:'2-digit',
        minute:'2-digit',
      }, //https://fullcalendar.io/docs/date-formatting
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
      this.calendarOptions.events.push({
        id:this.eventIndex++,
            start: e.start,
           end: e.end,
            display: 'auto',
            className:['bg-success'],
            editable:true
          });
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