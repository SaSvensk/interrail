<template>
    <div v-if="['STATION_CHANGE_PUBLIC_TRANSPORT', 'TRAIN_TRAVEL'].includes(leg.type)" style="display: flex; flex-direction: row; align-items: center; margin: 16px 0;">
        <div style="display: flex; justify-content: center; width: 70px;">
            <div>
                <div v-if="leg.type == 'TRAIN_TRAVEL'">
                    <!--<v-icon v-if="leg.transport && leg.transport.type == 'Ship'" icon="mdi-ferry" />
                    <v-icon v-else icon="mdi-train" />-->
                <div>
                {{ leg.transport.code }}
            </div>
                </div>
                <div v-else-if="leg.type == 'PLATFORM_CHANGE'">
                    <v-icon icon="mdi-transit-transfer" />
                </div>
                <div v-else-if="leg.type == 'STATION_CHANGE_PUBLIC_TRANSPORT'">
                    <v-icon icon="mdi-city-switch" />
                </div>
            </div>
           
        </div>
                    
        <div style="margin-left: 8px;">
            <div style="position: relative; display: flex; flex-direction: row; align-items: center;">
                <div style="z-index: 1; position: absolute; top: 50%; left: 6px; background-color: black; width: 8px; height: 50%;" /> 
                <div style="z-index: 2; width: 20px; height: 20px; background-color: white; border-radius: 50%; border: 2px solid black;" />
                <div style="margin-left: 8px;">
                    {{ leg.type == 'STATION_CHANGE_PUBLIC_TRANSPORT' ? '' : new Date(leg.start.dateTimeInISO).toLocaleTimeString('fi-FI', { hour: "2-digit", minute: "2-digit" }) }} 
                    {{ leg.start.station }}
                </div>
            </div>
            <div v-if="leg.type == 'TRAIN_TRAVEL' && leg.stops.length > 0" style="position: relative; display: flex; flex-direction: row; align-items: center;" @click="showStops = !showStops">
                <div style="position: absolute; left: 6px; background-color: black; width: 8px; height: 100%;" /> 

                <v-icon size="x-small" color="white" style="margin-left: 2.2px;" :icon="showStops ? 'mdi-menu-up' : 'mdi-menu-down'" />
                <div style="margin-left: 10px;">
                    {{ showStops ? 'Piilota' : 'Näytä' }} välipysähdykset
                </div>
            </div>
            <leg-stops v-show="showStops" :stops="leg.stops" />
            <div style="position: relative; display: flex; flex-direction: row; align-items: center;">
                <div style="z-index: 1; position: absolute; top: 0%; left: 6px; background-color: black; width: 8px; height: 50%;" /> 
                <div style="z-index: 2; width: 20px; height: 20px; background-color: white; border-radius: 50%; border: 2px solid black;" />
                <div style="margin-left: 8px;">
                    {{ leg.type == 'STATION_CHANGE_PUBLIC_TRANSPORT' ? '' : new Date(leg.end.dateTimeInISO).toLocaleTimeString('fi-FI', { hour: "2-digit", minute: "2-digit" }) }}
                    {{ leg.end.station }}
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import LegStops from './LegStops.vue'
export default {
    props: {
        leg: Object
    },
    components: { 
        LegStops
    },
    data() {
        return {
            showStops: false
        }
    }
}
</script>