<template>
    <v-hover v-slot="{ hover }">
        <v-list-item :key="item.id"
                     @click="(selectedMessage != undefined &&
                     item.id == selectedMessage.id) ?
                     setSelectedMessage(undefined) :
                     setSelectedMessage(item)">
            <v-list-item-content>
                <v-list-item-title v-text="item.from"></v-list-item-title>
                <v-list-item-subtitle
                        class="text--primary"
                        v-text="item.title"
                ></v-list-item-subtitle>
                <v-list-item-subtitle
                        v-if="item.content.length<40"
                        v-text="item.content">
                </v-list-item-subtitle>
                <v-list-item-subtitle v-else
                                      v-text="item.content.substring(0,40)+'...'">
                </v-list-item-subtitle>
            </v-list-item-content>
            <v-list-item-action>
                <v-list-item-action-text>
                    {{item.sent | customDate(item.sent)}}
                </v-list-item-action-text>
                <ActionButtons
                        v-bind:hover="hover"
                        v-bind:message="item"
                        v-bind:actions="actionButtonsConfig"
                />
            </v-list-item-action>
        </v-list-item>
    </v-hover>
</template>

<script>
import moment from 'moment';
import { mapState, mapMutations, mapActions } from 'vuex';
import ActionButtons from './ActionButtons.vue';

export default {
  name: 'Message',
  components: { ActionButtons },
  props: {
    item: Object,
  },
  data() {
    return {
      actionButtonsConfig: [
        {
          icon: () => 'replay',
          click: () => {},
          colorIcon: (favourite, color) => (color),
          parmanent: false,
        },
        {
          icon: () => 'delete',
          click: (message) => {
            this.$store.commit('addItemToList', { listName: 'trashboxMessages', message });
            this.$store.commit('removeItemByIdFromList', { listName: 'inboxMessages', message });
          },
          colorIcon: (favourite, color) => (color),
          parmanent: false,
        },
        {
          icon: favourite => (favourite ? 'star' : 'star_border'),
          colorIcon: favourite => (favourite ? 'yellow' : 'grey'),
          parmanent: true,
          click: () => this.toggleFavourite(this.item.id),
        },
      ],
    };
  },
  computed: {
    ...mapState(['selectedMessage']),
  },
  methods: {
    ...mapMutations(['setSelectedMessage']),
    ...mapActions(['toggleFavourite']),
  },
  filters: {
    customDate(value) {
      const currentTime = moment();
      const minutesDifference = parseInt(`${currentTime.diff(moment(value), 'minutes')}`, 0);
      const hoursDifference = parseInt(`${currentTime.diff(moment(value), 'hours')}`, 0);
      const daysDifference = parseInt(`${currentTime.diff(moment(value), 'days')}`, 0);

      let filterDate = '';

      if (daysDifference === 1) {
        filterDate = 'אתמול';
      } else if (daysDifference === 2) {
        filterDate = 'יומיים';
      } else if (daysDifference > 2 && daysDifference < 7) {
        filterDate = `${daysDifference.toString()} ימים`;
      } else if (daysDifference > 7) {
        filterDate = moment(value).format('DD.MM.YYYY');
      } else if (hoursDifference === 1) {
        filterDate = 'שעה';
      } else if (hoursDifference === 2) {
        filterDate = 'שעתיים';
      } else if (hoursDifference > 2) {
        filterDate = `${hoursDifference.toString()} שעות`;
      } else if (minutesDifference >= 0 && minutesDifference <= 1) {
        filterDate = 'דקה';
      } else {
        filterDate = `${minutesDifference.toString()} דקות`;
      }

      return filterDate;
    },
  },
};
</script>

<style scoped>
</style>
