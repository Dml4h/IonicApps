<template >
  <p
    class="time"
    v-text="currentTime"
  ></p>
</template>

<script>

import moment from 'moment';
export default{
    name: 'Clock',

    data () {
        return {
            currentTime: null
        }
    },
    methods: {
        updateCurrentTime () {
            this.currentTime = moment().format("h:mm a");
        }
    },
    created () {
        this.currentTime = moment().format("h:mm a");
        setInterval(() => this.updateCurrentTime());
    }
}
</script>

