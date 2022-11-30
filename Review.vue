<template lang="pug">
    .addresses-search
        h1.header Регионы

        input(
            v-model="region"
        )

        .addresses(
            v-if="addresses"
        )
            .address(
                v-for="(address, index) in addresses" 
                :key="index"
                :class="{'address-even': index % 2 === 0}"
            ) {{ address.name }}

        h2 Разделы

        .section(
            v-for="(section, index) in $t('sections')" 
            :key="index"
        ) {{ section }}
        
        .time {{ timeLimit }}
</template>

<script>
import { debounce } from 'lodash'

export default {
    name: 'addresses-search',
    watch: {
        region(newVal) {
            if (newVal.length > 1) {
                this.debouncedSearch()
            }
        }
    },
    data() {
        return {
            timer: null,
            region: '',
            addresses: [],
            timeLimit: 60,
        };
    },
    methods: {
        countDown() {
            this.timeLimit--;
            if (this.timeLimit <= 0) {
                this.stopTimer();
                this.$router.push('/');
            }
        },
        startTimer() {
            this.timer = setInterval(this.countDown, 1000);
        },
        stopTimer() {
            clearInterval(this.timer);
        },
        /**
         * @returns {Promise<void>}
         */
        async search() {
            this.addresses = await this.$api.FIAS.getAddress(this.region);
        },
    },
    created() {
        this.debouncedSearch = debounce(this.search, 100)
    },
    mounted() {
        this.startTimer()
    },
    beforeDestroy() {
        this.stopTimer()
    }
}
</script>

<style lang="sass">
.header
    font-size: 1.4rem

.address.address-even
    font-size: 18px
    font-weight: bold
</style>