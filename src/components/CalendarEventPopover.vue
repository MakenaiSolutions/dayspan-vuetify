<template>

  <v-card class="ds-calendar-event-popover-card"
    :class="classes">

   <v-toolbar extended flat
    :style="styleHeader">

     <v-toolbar-title slot="extension">
       {{ details.title }}
       <v-icon v-if="details.icon"
        :style="styleButton">
         {{ details.icon }}
       </v-icon>
     </v-toolbar-title>

     <slot name="eventPopoverToolbarLeft" v-bind="slotData"></slot>

     <v-spacer></v-spacer>

     <slot name="eventPopoverToolbarRight" v-bind="slotData"></slot>

     <slot name="eventPopoverToolbarActions" v-bind="slotData">

       <v-tooltip bottom v-if="!isReadOnly">

         <ds-schedule-actions
          slot="activator"
          v-bind="{$scopedSlots}"
          v-on="$listeners"
          :schedule="calendarEvent.schedule"
          :calendar-event="calendarEvent"
          :calendar="calendar">

          <v-btn icon :style="styleButton">
            <v-icon>more_vert</v-icon>
          </v-btn>

        </ds-schedule-actions>

        <span>{{ labels.options }}</span>

       </v-tooltip>

     </slot>

     <slot name="eventPopoverToolbarClose" v-bind="slotData">

       <v-btn icon @click="close" :style="styleButton">
         <v-icon>close</v-icon>
       </v-btn>

     </slot>

   </v-toolbar>
   <v-card-text>

     <slot name="eventPopoverBodyTop" v-bind="slotData"></slot>

     <v-list dense>

       <v-list-tile id="prueba">
         <v-list-tile-avatar >
           <v-icon>access_time</v-icon>
         </v-list-tile-avatar>
         <v-list-tile-content>
           <slot name="eventPopoverOccurs" v-bind="slotData" >
             <v-list-tile-title >{{ startDate }}</v-list-tile-title>
             <v-list-tile-sub-title>{{ occurs }}</v-list-tile-sub-title>
             <v-list-tile-sub-title v-if="hasDuration">{{ details.duration + ' ' + details.durationUnit }}</v-list-tile-sub-title>
           </slot>
         </v-list-tile-content>
       </v-list-tile>

       <v-list-tile v-if="details.location">
         <v-list-tile-avatar>
           <v-icon>location_on</v-icon>
         </v-list-tile-avatar>
         <v-list-tile-content>
           <slot name="eventPopoverLocation" v-bind="slotData">
             <v-list-tile-title>
               <span v-html="details.location"></span>
             </v-list-tile-title>
           </slot>
         </v-list-tile-content>
       </v-list-tile>

       <v-list-tile v-if="details.description">
         <v-list-tile-avatar>
           <v-icon>subject</v-icon>
         </v-list-tile-avatar>
         <v-list-tile-content>
           <slot name="eventPopoverDescription" v-bind="slotData">
             <v-list-tile-title>
               <span v-html="details.description"></span>
             </v-list-tile-title>
           </slot>
         </v-list-tile-content>
       </v-list-tile>

       <v-list-tile v-if="details.notify">
         <v-list-tile-avatar>
           <v-icon>notifications</v-icon>
         </v-list-tile-avatar>
         <v-list-tile-content>
           <slot name="eventPopoverNotifications" v-bind="slotData">
             <v-list-tile-title>
               <span v-html="details.notify"></span>
             </v-list-tile-title>
           </slot>
         </v-list-tile-content>
       </v-list-tile>

       <v-list-tile v-if="details.calendar">
         <v-list-tile-avatar>
           <v-icon>event</v-icon>
         </v-list-tile-avatar>
         <v-list-tile-content >
           <slot name="eventPopoverCalendar" v-bind="slotData">
             <v-list-tile-title>
               <span v-html="details.calendar"></span>
             </v-list-tile-title>
           </slot>
         </v-list-tile-content>
       </v-list-tile>

       <v-list-tile v-if="hasBusy">
         <v-list-tile-avatar>
           <v-icon>work</v-icon>
         </v-list-tile-avatar>
         <v-list-tile-content>
           <slot name="eventPopoverBusy" v-bind="slotData">
             <v-list-tile-title>{{ busyness }}</v-list-tile-title>
           </slot>
         </v-list-tile-content>
       </v-list-tile>

     </v-list>

     <slot name="eventPopoverBodyBottom" v-bind="slotData"></slot>

   </v-card-text>

   <slot name="eventPopoverActions" v-bind="slotData"></slot>

  </v-card>

</template>

<script>
import { CalendarEvent, Calendar, Pattern } from 'dayspan';


export default {

  name: 'dsCalendarEventPopover',

  props:
  {
    calendarEvent:
    {
      required: true,
      type: CalendarEvent
    },

    calendar:
    {
      required: true,
      type: Calendar
    },

    readOnly:
    {
      type: Boolean,
      default: false
    },

    edit:
    {
      type: Function
    },

    allowEditOnReadOnly:
    {
      type: Boolean,
      default() {
        return this.$dsDefaults().allowEditOnReadOnly;
      }
    },

    close:
    {
      type: Function
    },

    formats:
    {
      validate(x) {
        return this.$dsValidate(x, 'formats');
      },
      default() {
        return this.$dsDefaults().formats;
      }
    },

    labels:
    {
      validate(x) {
        return this.$dsValidate(x, 'labels');
      },
      default() {
        return this.$dsDefaults().labels;
      }
    }
  },

  computed:
  {
    slotData()
    {
      return {
        calendarEvent: this.calendarEvent,
        calendar: this.calendar,
        edit: this.edit,
        close: this.close,
        details: this.details,
        readOnly: this.readOnly
      };
    },

    classes()
    {
      return {
        'ds-event-cancelled': this.calendarEvent.cancelled
      };
    },

    styleHeader()
    {
      return {
        backgroundColor: this.details.color,
        color: this.details.forecolor
      };
    },

    styleButton()
    {
      return {
        color: this.details.forecolor
      };
    },

    startDate()
    {
      return this.calendarEvent.start.format( this.formats.start );
    },

    busyness()
    {
      return this.details.busy ? this.labels.busy : this.labels.free;
    },

    hasBusy()
    {
      return typeof this.details.busy === 'boolean';
    },

    hasDuration()
    {
      return this.details.duration != null;
    },

    occurs()
    {
      return this.$dayspan.getEventOccurrence(
        this.calendarEvent.schedule,
        this.calendarEvent.start,
        this.labels,
        this.formats
      );
    },

    details()
    {
      return this.calendarEvent.event.data;
    },

    allowEdit()
    {
      return this.allowEditOnReadOnly || !this.isReadOnly;
    },

    isReadOnly()
    {
      return this.readOnly || this.$dayspan.readOnly || this.details.readonly;
    }
  },

  data: vm => ({

  }),

  methods:
  {

  }
}
</script>
<style>
.v-list__tile {
   height: 100% !important;
}
</style>

<style scoped lang="scss">
.v-list{
  height:100%!important;
}
#prueba .v-list__tile {
        padding: 0px !important;
        height: 100% !important;
}
.ds-calendar-event-popover-card {

  .v-btn--floating.v-btn--left {
    margin-left: 0px !important;

    .v-icon {
      height: auto;
    }
  }

  .v-card__text {
    padding: 16px 0;

    .v-list {
      

      .v-list__tile {
        padding: 0px !important;
        height: 100% !important;

        .v-list__tile__sub-title {

        }
      }
    }
  }

  .v-list--dense{
  height: 100%;
}

  .v-toolbar__extension {
    padding: 0 16px !important;

    .v-toolbar__title {
      margin-left: 56px;
    }
  }
}

</style>
