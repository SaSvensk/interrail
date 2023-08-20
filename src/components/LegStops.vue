<template>
    <div style="margin-left: 5px;">
        <div 
            v-for="stop, stopIndex in stops" 
            v-bind:key="'stop-' + stopIndex"
            style="position: relative; display: flex; flex-direction: row; align-items: center;"
        >
            <div style="position: absolute; left: 1px; background-color: black; width: 8px; height: 100%;" /> 
            <div style="position: absolute; left: 3.5px; width: 4px; height: 4px; border-radius: 50%; background-color: white;" />
            <div style="margin-left: 22px">
                {{ timeString(stop) }} {{stop.station}} ({{stop.country}})
            </div>
            
        </div>
    </div>
</template>

<script>
export default {
    name: 'LegStops',
    props: {
        stops: Array
    },
    methods: {
        timeString(stop) {
            if (stop) {
                const hours = stop.time ? stop.time.hours > 9 ? stop.time.hours : '0' + stop.time.hours : '00'
                const minutes = stop.time ? stop.time.minutes > 9 ? stop.time.minutes : '0' + stop.time.minutes : '00'

                return hours + ':' + minutes
            }
            return 'nn:nn'
        }
    }
}
</script>