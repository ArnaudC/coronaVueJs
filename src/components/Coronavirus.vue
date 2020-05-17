<template>
    <div>
        <div class="coronavirus-latest" v-if="this.latest !== undefined">
            <div v-if="this.latest.deathRatio !== undefined" class="coronavirus-deathratio">
                Death ratio : {{deathRatioToFix}} %
            </div>
            <div>Total : {{this.latest.total}}</div>
            <div>Confirmed : {{this.latest.confirmed}}</div>
            <div>Deaths : {{this.latest.deaths}}</div>
            <div>Recovered : {{this.latest.recovered}}</div>
        </div>

        <div class="coronavirus-info">
            The list of countries below is sorted by the ratio (death) / (death + not death). On top, countries with the less death per infected.
        </div>

        <div class="coronavirus-holder" v-for="(l, index) in this.locations" :key="index">
            <h1 class="coronavirus-country">{{l.country}}{{l.province ? ' - ' + l.province : ''}}</h1>
                <div class="coronavirus-latest" v-if="l !== undefined">
                <div class="coronavirus-deathratio">
                    Death ratio : {{l.latest.deathRatioFixed}} %
                </div>
                <div>Total : {{l.latest.total}}</div>
                <div>Confirmed : {{l.latest.confirmed}}</div>
                <div>Deaths : {{l.latest.deaths}}</div>
                <div>Recovered : {{l.latest.recovered}}</div>
                <!--<div>Death ratio : {{l.latest.deathRatio}}</div>
                <div>Death ratio fixed : {{l.latest.deathRatioFixed}}</div>-->
            </div>
            <!--<LatestComponent :latest="this.location.latest"></LatestComponent>-->
        </div>
    </div>
</template>

<script>
export default {
    name: 'Coronavirus',
    data: function () { // fix error.
      return {
        locations: [],
        latest: [] // fix error.
      }
    },
    created () { // fetch the data when the view is created and the data is already being observed
        this.locations = [];
        this.fetchRemoteApi();
        // this.fetchAspnetcoreApi();
    },
    methods: {
        async fetchAspnetcoreApi() {
            let self = this;
            await fetch('https://localhost:5001/api/thisisanawsomenewdotnetcore32api/1')
            .then(res => res.json())
            .then(data => {
                self.corodata = data;
                self.latest = data?.latest ?? [];
                this.latest['total'] = (self.latest.confirmed || 0) + (self.latest.recovered || 0) + (self.latest.deaths || 0);
                let locationsOri = data?.locations ?? [];
                let res = this.getLocationsWithDeathRatio(locationsOri);
                self.locations = res;
                console.log('fetchRemoteApi: ', res);
            });
        },

        async fetchRemoteApi () {
            let self = this;
            await fetch('https://coronavirus-tracker-api.herokuapp.com/v2/locations')
            .then(res => res.json())
            .then(data => {
                self.corodata = data;
                self.latest = data?.latest ?? [];
                this.latest['total'] = (self.latest.confirmed || 0) + (self.latest.recovered || 0) + (self.latest.deaths || 0);
                let locationsOri = data?.locations ?? [];
                let res = this.getLocationsWithDeathRatio(locationsOri);
                self.locations = res;
                console.log('fetchRemoteApi: ', res);
            });
        },
        getLocationsWithDeathRatio(data) {
            let res = data
                .map(cor => {
                    let latestToUpdate = cor.latest;
                    cor.latest.total = (latestToUpdate.confirmed || 0) + (latestToUpdate.recovered || 0) + (latestToUpdate.deaths || 0);
                    cor.latest.deathRatio = (latestToUpdate.deaths || 0) / cor.latest.total * 100;
                    return cor;
                })
                .sort((a, b) => { // High ratio first
                    let r1 = a.latest.deathRatio || 0, r2 = b.latest.deathRatio || 0;
                    if (r1 < r2)
                        return -1;
                    if (r1 > r2)
                        return 1;
                    return 0;
                })
                .map(cor => {
                    let deathRatio = cor.latest.deathRatio;
                    cor.latest.deathRatioFixed = (deathRatio === undefined || typeof deathRatio !== 'number') ? '' : deathRatio.toFixed(2);
                    return cor;
                })
                .filter(cor => cor.latest.total >= 1000)
            ;
            return res;
        },
    }
}
</script>

<style>
.coronavirus-deathratio {
    font-size: 20px;
    margin-bottom: 7px;
}

.coronavirus-info {
    margin-top: 10px;
    font-size: 20px;
}
</style>
