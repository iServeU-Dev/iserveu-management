<template>
    <section class="cont">
        <controls-comp title="Εκκρεμότητες" path="ticketing" @searchFilter="setSearch"></controls-comp>
        <div class="filter">
            <div>Διευθετημένα:</div>
            <div class="icon_cont">
                <input type="checkbox" v-model="done" class="icon__check">
                <svg-icon class="icon icon__active" name="active" v-if="done"></svg-icon>
                <svg-icon class="icon icon__deactive" name="deactive" v-if="!done"></svg-icon>
            </div>
        </div>
        <div class="table">
            <ticket-entry v-for="ticket in tickets" :key="ticket.uid" :ticket="ticket"></ticket-entry>
        </div>
        <router-view></router-view>
    </section>
</template>
<script>
import Fuse from 'fuse.js'
import ControlsComp from './shared/ControlsComp.vue'
import TicketEntry from './tickets/TicketEntry.vue'
import SvgIcon from './shared/SvgIcon.vue'
export default {
    props:[
        "onSearchFilter"
    ],
    data() {
        return {
            searchFilter: ''
        }
    },
    components: {
        ControlsComp,
        TicketEntry,
        SvgIcon
    }, 
    computed: {
        tickets() {
            const ticks = this.$store.getters["ticket/tickets"]
            let returned= []
            const fuse = new Fuse(ticks, {
                keys: ['store.name', 'customer.name', 'customer.phone', 'customer.vat_number'],
                useExtendedSearch: true
            })
            if (this.searchFilter === "") {
                returned = ticks
            }else {
                let fuseSearch = fuse.search("'"+this.searchFilter)
                for (let i = 0; i < fuseSearch.length; i+=1) {
                    returned.push(fuseSearch[i].item)
                }
            }
            return returned.slice().sort((a,b) => {
                let fa = a.name.toLowerCase(), fb = b.name.toLowerCase();
                if (fa < fb) {
                    return -1
                }
                if (fa > fb) {
                    return 1
                }
                return 0
            })
        },
        done: {
            get() {
                return this.$store.getters["ticket/done"]
            },
            set(value) {
                this.$store.dispatch("ticket/setDone", value)
            }
        }
    },
    methods:{
        setSearch(value) {
            this.searchFilter = value
        }
    },
    mounted() {
        this.$store.dispatch("customer/loadCustomers")
        this.$store.dispatch("pending/loadPendings")
        if(this.$store.getters["ticket/done"] === false) {
            this.$store.dispatch("ticket/loadTickets")
        }
        else {
            this.$store.dispatch("ticket/loadDoneTickets")
        }
        
    }
}
</script>
<style scoped>
.filter {
        font-weight: bold;
        color: darkcyan;
        margin-top: 0.2rem;
        padding: 0.5rem;
        display: flex;
        align-items: center;
    }
    .icon_cont {
        cursor: pointer;
        margin-left: 0.2rem;
        display: grid;
        align-items: center;
    }
    .icon__check {
        z-index: 7;
        cursor: pointer;
        grid-column-start: 1;
        grid-row-start: 1;
        align-self: center;
        height: 1.5rem;
        width: 1.5rem;
    }
    .icon {
        grid-column-start: 1;
        grid-row-start: 1;
        align-self: center;
        height: 1.5rem;
        width: 1.5rem;
    }
    .icon__active {
        fill: var(--color-success);
    }
    .icon__deactive {
        fill: var(--color-danger);
    }
</style>