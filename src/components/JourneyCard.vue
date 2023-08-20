<template>
    <v-card style="margin: 4px;">
        <div style="margin: 12px;">
            <table>
                <tr v-for="terminus, index in termini" v-bind:key="'index-' + index + '-' + terminus.attribute">
                    <td>
                        {{ terminus.text }}
                    </td>
                    <td>
                        {{ item.legs[terminus.index][terminus.attribute].station }}
                    </td>
                    <td>
                        {{ new Date(item.legs[terminus.index][terminus.attribute].dateTimeInISO).toLocaleDateString('fi-FI') }},
                    </td>
                    <td>
                        klo {{ new Date(item.legs[terminus.index][terminus.attribute].dateTimeInISO).toLocaleTimeString('fi-FI', { hour: "2-digit", minute: "2-digit" }) }} 
                    </td>
                </tr>
            </table>
        </div>

        <div style="margin: 14px;">
            <div style="width: 100%; height: 40px; display: flex; flex-direction: row;">
                <div 
                    v-for="legP, index in legPortions" 
                    v-bind:key="'legv-' + index"
                    v-bind:style="{
                        'align-items': 'center',
                        'border': '1px solid gray',
                        'border-radius': '4px',
                        'display': 'flex',
                        'margin': '0 1px',
                        'overflow': 'hidden',
                        'white-space': 'nowrap',
                        'width': legP + '%'
                    }"
                >
                    {{ item.legs[index].transport ? item.legs[index].transport.code : '' }}
                </div>
            </div>
        </div>
        
            
        <div v-if="showLegs">
            <span style="margin-left: 14px;" @click="showLegs = false">
                Piilota etapit
            </span>
            <div v-for="leg, legIndex in item.legs" v-bind:key="'leg-' + legIndex">
                <journey-leg :leg="leg" />
                    <!--<v-slider
                        v-if="legIndex < item.legs.length - 1"
                        v-model="transferTime"
                        color="primary"
                        label="Vaihtoaika"
                    ></v-slider>-->
            </div>
            <span style="margin-left: 14px;" @click="showLegs = false">
                Piilota etapit
            </span>
        </div>
        <div v-else style="margin: 14px;">
            <span @click="showLegs = true">{{ item.legs.length > 0 ? item.legs.length - 1 + ' vaihtoa' : 'Suora' }}</span>
        </div>
    </v-card>
</template>

<script>
import JourneyLeg from './JourneyLeg.vue'
export default {
    props: {
        item: Object
    },
    components: {
        JourneyLeg
    },
    data() {
        return {
            showLegs: false,
            startIndex: 0
        }
    },
    computed: {
        endIndex() {
            return this.item.legs.length - 1
        },
        legPortions() {
            const sum = this.item.legs.reduce((a, b) => { return a + (b.duration.hours * 60) + b.duration.minutes}, 0)
            let tmp = []
            for (let i = 0; i < this.item.legs.length; i++) {
                tmp.push(this.countToMinutes(this.item.legs[i].duration) / sum * 100)
            }
            return tmp
        },
        termini() {
            return [
                {attribute: 'start', index: this.startIndex, text: 'Lähtö'},
                {attribute: 'end', index: this.endIndex, text: 'Saapuminen'}
            ]
        }
    },
    methods: {
        countToMinutes(obj = { hours: 0, minutes: 0}) {
            return (obj.hours) * 60 + obj.minutes
        }
    }
}
</script>