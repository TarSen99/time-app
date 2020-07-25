<template>
    <div>
        <h1 class="title">
            World Time
        </h1>

        <p class="error-msg" v-if="error">
            {{ error }}
        </p>

        <div class="loader" v-if="loading">
            Loading
        </div>

        <div class="time-content">
            <div class="field-block">
                <h4 class="section-title">
                    Select an area
                </h4>

                <select
                    v-model="area" 
                    class="default-select"
                >
                    <option disabled :value="null">Select an area</option>
                    <option
                        v-for="area in areas"
                        :key="area"
                        :value="area"
                    >
                        {{
                            area
                        }}
                     </option>
                </select>
            </div>

            <div class="field-block">
                <h4 class="section-title">
                    Select a location
                </h4>

                <select
                    :disabled="!locationSelectAvailable"
                    v-model="location" 
                    class="default-select"
                >
                    <option disabled :value="null">Select a location</option>
                    <option 
                        v-for="location in currentLocations"
                        :key="location"
                        :value="location"
                    >
                        {{
                            location
                        }}
                    </option>
                </select>
            </div>

            <div class="field-block">
                <h4 class="section-title">
                    The time in this location
                </h4>

                <h1 class="title time" v-if="time">
                    {{ time}}
                </h1>
                <p v-else>
                    Please select area and location
                </p>
            </div>

            <div class="reset-container">
                <button class="reset" @click="reset">
                    Reset the form
                </button>
            </div>
        </div>
    </div>
</template>

<script>
const axios = require('axios');
const BASE_URL = 'http://worldtimeapi.org/api'

const get = (url) => {
    return axios.get(`${BASE_URL}${url}`)
}

export default {
    name: 'WorldTime',
    data() {
        return {
            area: null,
            location: null,
            time: null,
            areas: [],
            locations: [],
            loading: true,
            error: null
        }
    },
    computed: {
        locationSelectAvailable() {
            return !!this.area
        },
        currentLocations() {
            return this.locations[this.area] || []
        }
    },
    watch: {
        area: {
            handler() {
                this.location = null
                this.time = null
            }
        },
        location: {
            handler(newV) {
                if(!newV) {
                    return
                }

                this.getTime(newV)
            }
        }
    },
    mounted() {
        this.error = null
        this.loading = true
        get('/timezone')
        .then(res => {
            this.loading = false
            const { data } = res

            const {areas, locations} = this.getBaseInfo(data)

            this.areas = areas;
            this.locations = locations;
        })
        .catch(err => {
                this.loading = false
                this.setError(err)
            })
    },
    methods: {
        reset() {
            this.area = null
            this.location = null
            this.error = null
        },
        getBaseInfo(zonesList) {
            const areas = []
            const locations = {}
            const splitBy = '/'

            zonesList.forEach(zone => {
                const area = zone.split(splitBy)[0]

                if(locations[area]) {
                    locations[area].push(zone)
                } else {
                    locations[area] = []
                    locations[area].push(zone)
                }

                if(areas.includes(area)) {
                    return
                }
                 
                areas.push(area)
            });

            return {areas, locations}
        },
        getTime(location) {
            this.error = null
            this.loading = true

            get(`/timezone/${location}`)
            .then(res => {
                this.loading = false
                const { data } = res
                this.time = this.parseTime(data.datetime, data.timezone)
            })
            .catch(err => {
                this.loading = false

                this.setError(err)
            })
        },
        parseTime(date, zone) {
            const dateNow = new Date(date)
            .toLocaleString("en", {timeZone: zone, hour: 'numeric', minute: 'numeric'})

            return dateNow
        },
        setError(err) {
            if(err && err.response && err.response.data && err.response.data.error) {
                this.error = err.response.data.error
            } else {
                this.error = 'Unexpected Error!'
            }
        }
    },
}
</script>

<style lang="scss" scoped>
    $radius: 5px;
    $main-color: #5F5784;
    $label-size: 14px;
    $title-size: 36px;
    $section-title-size: 14px;
    $select-size: 21px;
    $button-background: #79E1DE;

    button, select {
        outline: none;
    }

    .title {
        color: $main-color;
        font-size: $title-size;
        margin: 0;
    }

    .error-msg {
        padding: 10px;
        background-color: #FFF1E5;
        border-radius: $radius;
        color: #EE7E3C;
        text-align: center;
        font-weight: 900;
        font-size: $label-size;
    }

    .section-title {
        color: #605B79;
        font-size: $section-title-size;
        font-weight: 900;
        text-align: left;
        margin: 15px 0;
    }

    .default-select {
        border: 3px solid #8BD9D5;
        border-radius: $radius;
        width: 100%;
        height: 50px;
        font-size: $select-size;
        color: $main-color;
        padding-left: 10px;
    }

    .field-block {
        &:not(:first-of-type) {
            padding-top: 20px;
        }
    }

    .time {
        font-size: 48px;
        margin-top: 40px;
    }

    .reset-container {
        margin-top: 20px;
    }

    .reset {
        height: 40px;
        background-color: $button-background;
        color: $main-color;
        font-size: $label-size;
        font-weight: 900;
        border: none;
        border-radius: $radius;
        line-height: 40px;
        padding: 0 20px;
        margin: 0;
        cursor: pointer;
        transition: all 0.2s;

        &:hover {
            background-color: darken($button-background, 10%);
        }

        &:active {
            background-color: darken($button-background, 20%);
        }
    }

    .loader {
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        top: 10px;
    }
</style>