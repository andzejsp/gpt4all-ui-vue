<template>
    <div
        class="overflow-y-scroll flex flex-col no-scrollbar shadow-lg min-w-[24rem] max-w-[24rem] bg-bg-light-tone dark:bg-bg-dark-tone ">
        <!-- LEFT SIDE PANEL -->
        <div
            class="z-10 sticky top-0 flex-row p-2 flex items-center gap-3 flex-0 bg-bg-light-tone dark:bg-bg-dark-tone mt-0 px-4  shadow-md">
            <!-- CONTROL PANEL -->
            <div class=" text-2xl  hover:text-secondary duration-75 active:scale-90 " title="Create new discussion">
                <i data-feather="plus"></i>
            </div>
            <div class=" text-2xl  hover:text-secondary duration-75 active:scale-90 "
                title="Reset database, remove all discussions">
                <i data-feather="refresh-ccw"></i>
            </div>
            <div class=" text-2xl  hover:text-secondary duration-75 active:scale-90 " title="Export database">
                <i data-feather="database"></i>
            </div>
            <div class=" text-2xl  hover:text-secondary duration-75 active:scale-90 rotate-90"
                title="Export discussion to a file">
                <i data-feather="log-out"></i>
            </div>

            <!-- SEARCH BAR -->
            <form>
                <div class="relative">
                    <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                        <div class="scale-75 ">
                            <i data-feather="search"></i>
                        </div>
                    </div>
                    <div class="absolute inset-y-0 right-0 flex items-center pr-3 ">
                        <div class=" hover:text-secondary duration-75 active:scale-90 "
                            :class="filterTitle ? 'visible' : 'invisible'" title="Clear" @click="filterTitle = ''">
                            <i data-feather="x"></i>
                        </div>
                    </div>

                    <input type="search" id="default-search"
                        class="block w-full p-2 pl-10 pr-10 text-sm border border-gray-300 rounded-lg bg-bg-light focus:ring-secondary focus:border-secondary  dark:bg-bg-dark dark:border-gray-600 dark:placeholder-gray-400 dark:focus:ring-secondary dark:focus:border-secondary "
                        placeholder="Search..." title="Filter discussions by title" v-model="filterTitle"
                        @input="filterDiscussions()">
                </div>
            </form>
        </div>
        <div class="relative overflow-y-scroll no-scrollbar">
            <!-- DISCUSSION LIST -->
            <div class="mx-4 flex-grow" :class="filterInProgress ? 'opacity-20 pointer-events-none' : ''">

                <Discussion v-for="(item, index) in list" :key="index" :id="item.id" :title="item.title"
                    :selected="currentDiscussion.id == item.id" :loading="currentDiscussion.id == item.id && loading"
                    @click="selectDiscussion(item)" />

                <div v-if="list.length < 1"
                    class=" gap-2 py-2 my-2 hover:shadow-md hover:bg-primary-light dark:hover:bg-primary rounded-md p-2 duration-75 group cursor-pointer">
                    <p class="px-3">No discussions are found</p>
                </div>
                <div
                    class="sticky bottom-0  bg-gradient-to-t  pointer-events-none from-bg-light-tone dark:from-bg-dark-tone  flex height-64 ">
                    <!-- FADING DISCUSSION LIST END ELEMENT -->
                </div>
            </div>

        </div>

    </div>
    <div class="overflow-y-scroll flex flex-col no-scrollbar flex-grow "
        :class="loading ? 'opacity-20 pointer-events-none' : ''">
        <!-- CHAT AREA -->
        <div>
            <Message v-for="(msg, index) in discussionArr" :key="index" :message="msg"
                @click="scrollToElement($event.target)" />

            <WelcomeComponent v-if="discussionArr.length < 1" />

            <ChatBox v-if="discussionArr.length > 1" />

        </div>


    </div>
</template>
<style scoped>
.height-64 {
    min-height: 64px;
}
</style>

<script>
import axios from "axios";
import { nextTick } from 'vue'
export default {

    setup() {


    },
    data() {

        return {
            list: [],
            tempList: [],
            currentDiscussion: Number,
            discussionArr: [],
            loading: false,
            filterTitle: "",
            filterInProgress: false
        }
    },
    methods: {
        async list_discussions() {
            try {
                const res = await axios.get("/api/list_discussions");

                if (res) {

                    return res.data

                }
            } catch (error) {
                console.log(error)
                return []
            }


        },
        async load_discussion(id) {
            try {


                if (id) {
                    this.loading = true
                    const res = await axios.post("/api/load_discussion", {
                        id: id
                    });

                    if (res) {
                        this.discussionArr = res.data
                        this.loading = false
                    }
                }

            } catch (error) {
                console.log(error)
                this.loading = false
            }

        },
        filterDiscussions() {
            if (!this.filterInProgress) {
                this.filterInProgress = true
                setTimeout(() => {

                    this.list = this.tempList.filter((item) => item.title.includes(this.filterTitle))
                    this.filterInProgress = false

                }, 100)
            }
        },
        selectDiscussion(item) {
            this.currentDiscussion = item
            this.load_discussion(item.id)

        },
        scrollToElement(el) {


            if (el) {

                el.scrollIntoView({ behavior: 'smooth', block: "center", inline: "nearest" });
            }
        }
    },
    async mounted() {
        this.list = await this.list_discussions()
        this.tempList = this.list
        nextTick(() => {
            feather.replace()

        })
    }, components: {
        Discussion,
        Message,
        ChatBox,
        WelcomeComponent
    }, watch: {
        filterTitle(newVal, oldVal) {
            if (newVal == "") {
                this.filterInProgress = true
                this.list = this.tempList
                this.filterInProgress = false
            }
        }
    }

}
</script>

<script setup >
import Discussion from '../components/Discussion.vue';
import Message from '../components/Message.vue';
import ChatBox from '../components/ChatBox.vue'
import WelcomeComponent from '../components/WelcomeComponent.vue'
import feather from 'feather-icons'
</script>