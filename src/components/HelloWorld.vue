
<template>
<v-container fluid>
    <v-row>
        <v-col>
            <v-btn @click="createMap">
                Create map
            </v-btn>
        </v-col>
    </v-row>
    <v-row>
    <v-col class="greetings">
        <v-autocomplete
            v-model="startStation"
            v-model:search="startPointText"
            label="Lähtö Suomessa"
            :hide-no-data="startPointText && !loading"
            item-title="station"
            :items="items"
            return-object
            @click="items = []"
        />
        <v-autocomplete
            v-model="endStation"
            v-model:search="endPointText"
            :hide-no-data="endPointText && !loading"
            label="Määränpää"
            item-title="station"
            :items="items"
            return-object
            @click="items = []"
        />
        <v-row>
            <v-col style="width: 100px;">
                <v-text-field label="Minimi vaihtoaika" />
            </v-col>
        </v-row>
        <v-row>
            <v-col class="greetings">
                <v-row class="greetings">
                    <v-container fluid>
                        <v-row>
                            <v-col>
                                <v-text-field v-model="startTime" :disabled="selectedOption == 3" label="Alkaa" />
                            </v-col>
                            <v-col>
                                <v-text-field v-model="endTime" :disabled="selectedOption == 3" label="Loppuu" />
                            </v-col>
                        </v-row>
                        <v-row>
                            <v-radio-group v-model="selectedOption">
                                <v-radio label="Ei matkantekoa ajan ulkopuolella" value="1"></v-radio>
                                <v-radio label="Max 1 yhteys joka kestää koko ulkopuolisen aikavälin" value="2"></v-radio>
                                <v-radio label="Ei rajoituksia. Matkantekoa 24/7!!" value="3"></v-radio>
                            </v-radio-group>
                        </v-row>
                    </v-container>
                </v-row>
            </v-col>
        </v-row>
    </v-col>
    <v-col>
        <div v-if="journeysLoading">
            <v-progress-circular
                indeterminate
                color="primary"
            ></v-progress-circular>
        </div>
        
        <journey-card 
            v-for="(item, journeyIndex) in journeys" 
            v-bind:key="'journey-' + journeyIndex" 
            :item="item"
        />
        
    </v-col>
    </v-row>
</v-container>
</template>
<script>
import axios from 'axios'
import JourneyCard from './JourneyCard.vue'
export default {
    name: 'Search',
    components: { 
        JourneyCard 
    },
    data() {
        return {
            endPointText: '',
            endStation: null,
            endTime: '22:00',
            items: [],
            journeys: [],
            journeysLoading: false,
            loading: false,
            selectedOption: '1',
            startPointText: null,
            startStation: null,
            startTime: '06:00',
            transferTime: 5,
        }
    },
    watch: {
        endPointText(v) {
            this.items = []
            this.loading = true
            this.loadLocations(v, false).then(r => {
                this.items = r
            })
            .finally(() => {
                this.loading = false
            })
        },
        selectedOption() {
            if (this.startStation && this.endStation) {
                this.startConnectionBuild(this.startStation.id, this.endStation.id)
            }
        },
        startPointText(v) {
            this.items = []
            this.loading = true
            this.loadLocations(v).then(r => {
                const regExp = /\(([^)]+)\)/;
                this.items = r.filter(x => regExp.exec(x.station)[1] == 'Finland').map(x => x = {...x, station: x.station.split('(')[0].trim()})
            })
            .finally(() => {
                this.loading = false
            })
        },
        endStation(v = {}) {
            if (v.id && this.startStation && this.startStation.id) {
                
                this.startConnectionBuild(this.startStation.id, v.id)
            }
        },
        startStation(v = {}) {
            if (v.id && this.endStation && this.endStation.id) {
                this.startConnectionBuild(v.id, this.endStation.id)
            }
        }
    },
    methods: {
        addLeg(arr = [], leg) {
            if (leg.type != 'PLATFORM_CHANGE') {
                arr.push(leg)
            }
        },
        loadMapConnection(arr = []) {
            for (let i = 0; i < arr.length; i++) {
                console.log(i + 1)
                for (let j = 0; j < arr[i].legs.length; j++) {
                    if (arr[i].legs[j].start.station != arr[i].legs[j].end.station) {
                        console.log(arr[i].legs[j].start.station, '-', arr[i].legs[j].end.station)
                    }
                }
            }
        },
        createMap() {
            //hamburg: 008002710

            const connections = [
                {start: '007600100', end: '008002710', name: 'Oslo - Hamburg'},
                {start: '009900009', end: '008002710', name: 'Stockholm - Hamburg'},
                {start: '007601126', end: '008002710', name: 'Trondheim - Hamburg'},
                {start: '001000130', end: '009900009', name: 'Turku - Stockholm'},
                {start: '007404337', end: '009900009', name: 'Umeå - Hamburg'}
            ]

            let p = []

            for (let i = 0; i < connections.length; i++) {
                p.push(this.loadSimpleConnection(connections[i].start, connections[i].end))
            }

            Promise.all(p).then(r => {
                for (let i = 0; i < connections.length; i++) {
                    this.loadMapConnection(r[i])
                }
            })
        },
        startConnectionBuild(startId, endId) {
            if (startId && endId) {
                this.journeysLoading = true
                this.journeys = []
                this.loadConnections(startId, endId).then(r => {
                    console.log(r)
                    this.journeys = r
                })
                .finally(() => {
                    this.journeysLoading = false
                })
            }
        },
        checkThatJourneysAreComplete(v = []) {
            return new Promise((resolve, reject) => {
                let tmp = []
                let indices = []
                if (this.startStation && this.endStation) {
                    //let journeyIds = []
                    
                    for (let i = 0; i < v.length; i++) {
                        const journeyEndId = v[i].legs[v[i].legs.length - 1].end.id
                        if (journeyEndId && journeyEndId != this.endStation.id) {

                            const e = '00' + journeyEndId
                            //if (!journeyIds.includes(e)) {
                                
                            //journeyIds.push(e)
                            tmp.push(Promise.resolve(e))
                            indices.push(i)
                            //}
                        } else if (!journeyEndId) {
                            indices.push(i)
                            tmp.push(this.loadLocations(v[i].legs[v[i].legs.length - 1].end.station, false).then(r => {
                                return r[0].id
                            }))
                        }
                    }
                }
                Promise.all(tmp).then(r => {
                    let idsAndTimeStamps = []
                    for (let i = 0; i < v.length; i++) {

                        const journeyEndId = v[i].legs[v[i].legs.length - 1].end.id

                        //define start date
                        const startDate = v[i].legs[v[i].legs.length - 1].end.dateTimeInISO.split('T')
                        let d = new Date(startDate);
                        d.setUTCDate(d.getUTCDate() + 1);
                        const timeStamp = d.toISOString().split('T')[0] + 'T' + this.startTime.replaceAll(':', '%3A') + '%3A00.000Z'

                        const strippedEndStationId = this.endStation.id.slice(2)

                        if (!journeyEndId || (journeyEndId && journeyEndId != this.endStation.id && journeyEndId != strippedEndStationId)) {
                            idsAndTimeStamps.push({id: r[i], timeStamp, index: indices.pop()})
                        }
                    }
                    resolve(idsAndTimeStamps)
                })
                .catch(e => {
                    reject(e)
                })
            })
            
        },
        getMatchingJourneys(j = [], option) {
            const startMinutes = Number(this.startTime.split(':')[0]) * 60 + Number(this.startTime.split(':')[1])
            const endMinutes = Number(this.endTime.split(':')[0]) * 60 + Number(this.endTime.split(':')[1])

            let tmp = []
            for (let i = 0; i < j.length; i++) {
                const journey = JSON.parse(JSON.stringify(j[i]))
                let updatedLegs = []
                for (let j = 0; j < journey.legs.length; j++) {
                    const leg = journey.legs[j]

                    const legStartTime = new Date(leg.start.dateTimeInISO).toLocaleTimeString('fi-FI', { hour: "2-digit", minute: "2-digit" })
                    const legEndTime = new Date(leg.end.dateTimeInISO).toLocaleTimeString('fi-FI', { hour: "2-digit", minute: "2-digit" })
                    const legStartMinutes = Number(legStartTime.split('.')[0]) * 60 + Number(legStartTime.split('.')[1])
                    const legEndMinutes = Number(legEndTime.split('.')[0]) * 60 + Number(legEndTime.split('.')[1])

                    //leg is in the given time period
                    //e.g. 9-14 is in time period 8-22
                    if (option == 3) {
                        this.addLeg(updatedLegs, leg)
                    } else {

                        //case e.g
                        //start time: 6.00, end time: 22.00
                        //leg start: 7.00, leg end: 11.00
                        if (legStartMinutes >= startMinutes && legEndMinutes <= endMinutes && legEndMinutes >= startMinutes) {
                            if (legStartMinutes > legEndMinutes) {
                                if (option == 2) {
                                    this.addLeg(updatedLegs, leg)
                                } else {
                                    break;
                                }
                            } else {
                                this.addLeg(updatedLegs, leg)
                            }
                        } else if (option != 1 || (option == 1 && legStartMinutes >= startMinutes && legEndMinutes <= endMinutes)) {
                            if (legStartMinutes < endMinutes) {
                                let remainingStops = []
                                if (leg.stops) {
                                    for (let k = 0; k < leg.stops.length; k++) {
                                        if (leg.stops[k].time) {
                                            const stopMinutes = (leg.stops[k].time.hours * 60) + leg.stops[k].time.minutes
                                            if (stopMinutes <= endMinutes) {
                                                remainingStops.push(leg.stops[k])
                                            } else {
                                                break;
                                            }
                                        }                                 
                                    }
                                }

                                //update leg stops.
                                if (remainingStops.length > 0) {
                                    if (option != 3) {
                                        if (legEndMinutes < startMinutes) {
                                            leg.end = {
                                                country: remainingStops[remainingStops.length - 1].country,
                                                dateTimeInISO: leg.start.dateTimeInISO.split('T')[0] + 'T' + remainingStops[remainingStops.length - 1].time.hours + ':' + remainingStops[remainingStops.length - 1].time.minutes + ':00',
                                                station: remainingStops[remainingStops.length - 1].station,
                                                time: remainingStops[remainingStops.length - 1].time
                                            }
                                            remainingStops.pop()
                                        }
                                    }
                                    leg.stops = remainingStops
                                    
                                    this.addLeg(updatedLegs, leg)
                                }
                            }
                            break;
                        } else {
                            break;
                        }
                    }
                }

                for (let j = updatedLegs.length - 1; j >= 0; j--) {
                    if (updatedLegs[j].type != 'TRAIN_TRAVEL') {
                        updatedLegs.splice(j, 1)
                    } else {
                        break;
                    }
                }

                journey.legs = updatedLegs
                if (journey.legs && journey.legs.length > 0) {
                    tmp.push(journey)
                }
                
            }

            return tmp
        },
        loadSimpleConnection(startId = '', endId = '', timeStamp = new Date().toISOString().replaceAll(':', '%3A'), limit = 5) {
            return new Promise((resolve, reject) => {
                if (startId && endId) {
                    /*
                    params: minChangeTime (number)
                            timeStamp (ISO DateTime) 2023-07-16T14%3A00%3A00.000Z
                    */

                    axios.get(
                        'https://api.timetable.eurail.com/v2/timetable?origin=' + startId + '&destination=' + endId + '&via=&timestamp=' + timeStamp + '&tripsNumber=' + limit + '&arrival=false&currency=EUR&travellers=1'
                    ).then(r => {
                        resolve(r)
                    })
                    .catch(e => {
                        reject(e)
                    })
                }
            })
        },
        loadConnections(startId = '', endId = '', timeStamp = new Date().toISOString().replaceAll(':', '%3A'), limit = 5) {

            return new Promise((resolve, reject) => {
                if (startId && endId) {
                    /*
                    params: minChangeTime (number)
                            timeStamp (ISO DateTime) 2023-07-16T14%3A00%3A00.000Z

                    */

                    this.loadSimpleConnection(startId, endId, timeStamp, limit).then(r => {
                        let p = []
                        for (let i = 0; i < r.data.length; i++) {
                            p.push(this.loadJourneyStops(r.data[i], r.data[i].legs))
                        }

                        Promise.all(p).then(r => {

                            //create the journeys so that they include only legs valid for traveling options
                            const updatedJourneys = this.getMatchingJourneys(r, this.selectedOption)

                            //check which journeys are complete
                            this.checkThatJourneysAreComplete(updatedJourneys).then(unfinishedJourneys => {

                                //if there are no unfinished journeys, stop processing
                                if (unfinishedJourneys.length == 0) {
                                    resolve(updatedJourneys)
                                } else {

                                    //else, start ir from the latest stop
                                    let p = []
                                    for (let i = 0; i < unfinishedJourneys.length; i++) {
                                        p.push(this.loadConnections(unfinishedJourneys[i].id, endId, unfinishedJourneys[i].timeStamp, 1, level + 1))
                                    }

                                    Promise.all(p).then(transfers => {
                                        for (let i = 0; i < updatedJourneys.length; i++) {
                                            const j = unfinishedJourneys.findIndex(x => x.index == i)
                                            if (j > -1) {
                                                updatedJourneys[j].legs = updatedJourneys[j].legs.concat(transfers[j][0].legs)
                                            }
                                        }
                                        resolve(updatedJourneys)
                                    })
                                }
                            })
                        })
                    })

                //Göteborg C: 740000002
                //Haparanda: 740000372
                //Halmstad C: 740000080
                //Malmö C: 740000003
                //Nynäshamn: 740000727
                //Stockholm C: 740000001
                //Trelleborg: 740000088
                //Umeå: 740020461

                //ruotsi-apikey: e276bfc7-eaeb-4bc0-82d2-ec8fd2553630
                //get stop by name: https://api.resrobot.se/v2.1/location.name?input=G%C3%B6teborg&format=json&accessId=API_KEY
                //get route: https://api.resrobot.se/v2.1/trip?format=json&originId=stop.extId&destId=stop.extId&passlist=true&showPassingPoints=true&accessId=API_KEY

                //Yhteydet
                //1. Haetaan Interrail-apista suora yhteys Suomi-kohde - Ulkomaan kohde (säilytä tämä)
                //2. Haetaan tiedostosta laiva-yhteys Nynäshamn ja Uumajaan (vain alussa)
                //3. Haetaan Resrobot-apista myös yhteys Tukholma/Nynäshamn - Göteborg/Karlskrona/Trelleborg
                //4. Haetaan Interrail-apista jatkoyhteydet alk. Kiel, Gdynia, Rostock (kun tiedetään päivä)
                //-----
                //Katsotaan mihin asti matkustus ehtii -->
                //jos ei rajoitusta, käytetään kohdassa 1 haettua yhteyttä (jos löytynyt, jos ei, hae uudestaan jo saavutetusta kohdasta)
                //jos rajoitus (ei), katkaise matkanteko viimeisimpään paikkaan joka täyttää aikarajan
                //jos rajoitus (yksi yhteys), hae yhteys joka lähtee/saapuu min. vaihtoajan puitteissa tai jatkuu aikarajan yli.
                //Toista kunnes määränpää saavutettu.      
                
                //Esim. Tampere - Hampuri
                //hae Interrail - Tampere-Hampuri
                //Hae myös 

                //FRGO - Fredrikshavn (DEN) - Göteborg (SWE)
                //GDKA - Gdynia (PL) - Karlskrona (SWE)
                //GOKI - Göteborg (SWE) - Kiel (DE)
                //GRHA - Grenå (DEN) - Halmstad (SWE)
                //HFNY - Hanko (FIN) - Nynäshamn (SWE)
                //RSTB - Rostock (DE) - Trelleborg (SWE)
                //url: https://www.stenaline.fi/reittimme/_jcr_content.timetable.[ROUTE_CODE].[YYYY-MM-DD].json
            } else {
                reject()
            }

            })
            
        },
        loadLocations(v = '') {
            return new Promise((resolve, reject) => {
                if (v) {
                    axios.get(
                        'https://api.timetable.eurail.com/v2/locations?input=' + v + '&results=15'
                    ).then(r => {
                        resolve(r.data)
                    })
                    .catch(() => {
                        reject()
                    })
                } else {
                    resolve([])
                }
            })
            
        },
        loadJourneyStops(journey = {}, legs = []) {
            return new Promise((resolve, reject) => {
                let p = []
                for (let i = 0; i < legs.length; i++) {
                    if (legs[i].type == "TRAIN_TRAVEL") {
                        const id = legs[i].id.replaceAll('|', '%7C')
                        p.push(axios.get('https://api.timetable.eurail.com/v2/stops?leg=' + id + '&start=' + legs[i].start.id + '&end=' + legs[i].end.id).then(r => {
                            return r.data
                        }))
                    }
                }

                Promise.all(p).then(r => {
                    const endHours = Number(this.endTime.split(':')[0])
                    const startHours = Number(this.startTime.split(':')[0])

                    let index = 0
                
                    for (let i = 0; i < legs.length; i++) {
                     
                        if (legs[i].type == "TRAIN_TRAVEL") {
                            legs[i].stops = r[index] ? r[index].stops ? r[index].stops : [] : []
                            index = index + 1
                        }
                    }
                    journey.legs = legs
                    resolve(journey)
                })
                .catch(() => {
                    reject()
                })
            })
            
            
        }
    }
}
</script>

<style scoped>

</style>
