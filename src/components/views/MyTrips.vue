<template>
  <div class="trips container">
        <div class="col-xs-24">
            <Loading :data="pendingRequest" :hideOnEmpty="true">
                <h2 slot="title"> Pendientes <strong>de contestar</strong> </h2>
                <div class="request-list">
                    <PendingRequest v-for="r in pendingRequest" :user="r.user" :trip="findTrip(r.trip_id)"></PendingRequest>
                </div>
                <p slot="no-data" class="alert alert-warning"  role="alert">No hay pedientes de contestar</p>
                <p slot="loading" class="alert alert-info" role="alert">
                    <img src="https://carpoolear.com.ar/static/img/loader.gif" alt="" class="ajax-loader" />
                    Cargando...
                </p>
            </Loading>
        </div>

        <div class="col-xs-24">
            <Loading :data="pendingRates" :hideOnEmpty="true">
                <h2 slot="title"> Calificaciones <strong>pendientes </strong></h2>
                <div class="request-list">
                    <RatePending v-for="rate in pendingRates" :rate="rate" />
                </div>
                <p slot="no-data" class="alert alert-warning"  role="alert">No hay calificaciones pendientes</p>
                <p slot="loading" class="alert alert-info" role="alert">
                    <img src="https://carpoolear.com.ar/static/img/loader.gif" alt="" class="ajax-loader" />
                    Cargando calificaciones ...
                </p>
            </Loading>
        </div>

        <div class="col-xs-24">
            <h2>Mis <strong>próximos</strong> viajes</h2>
            <Loading :data="trips">
                <div class="trips-list">
                    <Trip v-for="trip in trips" :trip="trip" :user="user" :enableChangeSeats="true"></Trip>
                </div>
                <p slot="no-data" class="alert alert-warning"  role="alert">No tenés viajes creados</p>
                <p slot="loading" class="alert alert-info" role="alert">
                    <img src="https://carpoolear.com.ar/static/img/loader.gif" alt="" class="ajax-loader" />
                    Cargando viajes ...
                </p>
            </Loading>
        </div>

        <div class="col-xs-24">
            <Loading :data="passengerTrips" :hideOnEmpty="true">
                <h2 slot="title" > Viajes a los que <strong>estoy subido</strong> </h2>
                <div class="trips-list">
                    <Trip v-for="trip in passengerTrips" :trip="trip" :user="user"></Trip>
                </div>
                <p slot="no-data" class="alert alert-warning"  role="alert">No estas subido a ningún viaje.</p>
                <p slot="loading" class="alert alert-info" role="alert">
                    <img src="https://carpoolear.com.ar/static/img/loader.gif" alt="" class="ajax-loader" />
                    Cargando viajes ...
                </p>
            </Loading>
        </div>
        <div class="col-xs-24" v-if="subscriptions && subscriptions.length" id="suscriptions">
            <Loading :data="subscriptions" :hideOnEmpty="true">
                <h2 slot="title" > Suscripciones a viajes</h2>
                <div class="trips-list row">
                    <div class="col-xs-24 col-md-12" v-for="subs in subscriptions" :key="subs.id">
                        <subscriptionItem :subscription="subs" :user="user"></subscriptionItem>
                    </div>

                </div>
                <p slot="no-data" class="alert alert-warning"  role="alert">No tienes ninguna suscripción.</p>
                <p slot="loading" class="alert alert-info" role="alert">
                    <img src="https://carpoolear.com.ar/static/img/loader.gif" alt="" class="ajax-loader" />
                    Cargando suscripciones ...
                </p>
            </Loading>
        </div>


        <div class="col-xs-24" v-if="oldTrips">
            <h2>Mis viajes pasados</h2>
            <Loading :data="oldTrips">
                <div class="trips-list">
                    <Trip v-for="trip in oldTrips" :trip="trip" :user="user"></Trip>
                </div>
                <p slot="no-data" class="alert alert-warning"  role="alert">No create ningún viaje</p>
                <p slot="loading" class="alert alert-info" role="alert">
                    <img src="https://carpoolear.com.ar/static/img/loader.gif" alt="" class="ajax-loader" />
                    Cargando viajes ...
                </p>
            </Loading>
        </div>

        <div class="col-xs-24" v-if="oldPassengerTrips">
            <Loading :data="oldPassengerTrips" :hideOnEmpty="true">
                <h2 slot="title" > Viajes a los que me <strong>subí</strong> </h2>
                <div class="trips-list">
                    <Trip v-for="trip in oldPassengerTrips" :trip="trip" :user="user"></Trip>
                </div>
                <p slot="no-data" class="alert alert-warning"  role="alert">No te has subido a ningún viaje.</p>
                <p slot="loading" class="alert alert-info" role="alert">
                    <img src="https://carpoolear.com.ar/static/img/loader.gif" alt="" class="ajax-loader" />
                    Cargando viajes ...
                </p>
            </Loading>
        </div>

    </div>
</template>

<script>
import subscriptionItem from '../sections/SubscriptionItem.vue';
import Trip from '../sections/Trip.vue';
import Loading from '../Loading.vue';
import PendingRequest from '../PendingRequest';
import RatePending from '../RatePending';
import { mapGetters, mapActions } from 'vuex';

import Tab from '../elements/Tab';
import Tabset from '../elements/Tabset';

export default {
    name: 'my-trips',

    mounted () {
        this.tripAsDriver();
        this.tripAsPassenger();
        this.pendingRate();
        this.getPendingRequest().then(() => {
            this.oldTripsAsDriver();
            this.oldTripsAsPassenger();
        });
        this.findSubscriptions();
    },
    computed: {
        ...mapGetters({
            trips: 'myTrips/myTrips',
            passengerTrips: 'myTrips/passengerTrips',
            pendingRates: 'rates/pendingRates',
            pendingRequest: 'passenger/pendingRequest',
            user: 'auth/user',
            oldTrips: 'myTrips/myOldTrips',
            oldPassengerTrips: 'myTrips/passengerOldTrips',
            subscriptions: 'subscriptions/subscriptions'
        })
    },

    methods: {
        findTrip (id) {
            if (this.trips) {
                return this.trips.find(item => item.id === id);
            }
        },
        updateScroll () {
            if (this.$route.query.loc) {
                // window.scrollTo(0, document.body.scrollHeight);
                // window.location.hash = this.$route.query.loc;
                let domNode = document.getElementById(this.$route.query.loc);
                window.scrollTo(0, domNode.offsetTop - 150);
            }
        },
        ...mapActions({
            tripAsDriver: 'myTrips/tripAsDriver',
            tripAsPassenger: 'myTrips/tripAsPassenger',
            pendingRate: 'rates/pendingRates',
            getPendingRequest: 'passenger/getPendingRequest',
            oldTripsAsDriver: 'myTrips/oldTripsAsDriver',
            oldTripsAsPassenger: 'myTrips/oldTripsAsPassenger',
            findSubscriptions: 'subscriptions/index'
        })
    },
    watch: {
        trips: function () {
            this.updateScroll();
        },
        passengerTrips: function () {
            this.updateScroll();
        },
        pendingRates: function () {
            this.updateScroll();
        },
        pendingRequest: function () {
            this.updateScroll();
        },
        user: function () {
            this.updateScroll();
        },
        oldTrips: function () {
            this.updateScroll();
        },
        oldPassengerTrips: function () {
            this.updateScroll();
        }/* ,
        subscriptions: function () {
            this.updateScroll();
        } */
    },
    components: {
        Trip,
        Loading,
        PendingRequest,
        RatePending,
        Tab,
        Tabset,
        subscriptionItem
    }
};
</script>

<style scoped>
    h2 {
        font-weight: 300;
    }
</style>
