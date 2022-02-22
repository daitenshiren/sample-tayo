<template>
  <div>
    <div class="content__wrapper pods sidebarOpenRight py-0">
      <div class="container-fluid h-100">
        <div class="row h-100">
          <div class="pod-sidebar-box">
            <div class="pod-sidebar">
              <div class="left__section-content">
                <div class="milestone-left">
                  <pods-icons-list/>
                  <mile-stone-list
                      @openConversation="openConversationModule()"
                      @openSubscription="openSubscriptionModule()"
                      @openPodsChat="openPodsModule()"
                  />
                </div>
              </div>
            </div>
          </div>
          <div class="pod-main-content">
            <div :class="'chatContainer'">
              <div class="card group_chatbox border-0">
                <section v-if="milestone" class="milestone pods py-0" style="padding-right: 0 !important">
                  <milestone-details :milestone="milestone" @change="OnChangeMilestone"/>
                </section>

                <div v-if="!milestone">
                  <div class="card-header border-0 d-flex justify-content-between align-items-center">
                    <div class="d-flex align-items-center">
                      <ul class="groupUserList">
                        <template v-for="(item, i) in selectedPod.user">
                          <li v-if="i < 2" :key="i">
                            <img src="@/assets/images/feed-avatar3.png" class="img-fluid" alt=""/>
                          </li>
                        </template>
                        <li v-if="selectedPod && selectedPod.user.length > 2">
                          <span>{{ selectedPod.user.length - 2 }}+</span>
                        </li>
                      </ul>
                      <div>
                        <h6 class="pl-2" v-if="openConversation">Conversations</h6>
                        <h6 class="pl-2" v-else>Subscription Feed</h6>
                      </div>
                    </div>
                    <div class="d-flex align-items-center">
                      <a
                          data-bs-toggle="modal"
                          data-bs-target="#addNewMumber"
                          @click="openManageMembers = true"
                          class="nav-link pe-0 me-2 m-3 my-2"
                      >
                        <svg xmlns="http://www.w3.org/2000/svg" width="1.219rem" height="1.219rem" viewBox="0 0 19.5 19.5">
                          <g transform="translate(0.75 0.75)">
                            <path
                                class="addUserA"
                                v-bind:style="{ stroke: [$store.state.isDarkMode ? '#9da4ac' : '#363853'] }"
                                d="M21,12H17m2,2V10"
                                transform="translate(-3 -3)"
                            ></path>
                            <path
                                class="addUserb"
                                v-bind:style="{ stroke: [$store.state.isDarkMode ? '#9da4ac' : '#363853'] }"
                                d="M3,19.111a4.865,4.865,0,0,1,4-4.849l.208-.034a17.134,17.134,0,0,1,5.576,0l.208.034a4.865,4.865,0,0,1,4,4.849A1.859,1.859,0,0,1,15.172,21H4.828A1.859,1.859,0,0,1,3,19.111Z"
                                transform="translate(-3 -3)"
                            ></path>
                            <path
                                class="addUserb"
                                v-bind:style="{ stroke: [$store.state.isDarkMode ? '#9da4ac' : '#363853'] }"
                                d="M14.083,6.938A4.012,4.012,0,0,1,10,10.875,4.012,4.012,0,0,1,5.917,6.938,4.012,4.012,0,0,1,10,3,4.012,4.012,0,0,1,14.083,6.938Z"
                                transform="translate(-3 -3)"
                            ></path>
                          </g>
                        </svg>
                      </a>
                    </div>
                  </div>
                  <manage-members v-show="openManageMembers" @close="close()"/>
                  <subscription-feed v-show="openSubscriptionFeed"/>
                  <conversation v-show="openConversation"/>
                  <pods-chat v-show="openPods"/>
                </div>
              </div>
            </div>
          </div>
          <pod-right-side-menu/>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import {mapActions, mapGetters} from 'vuex';
  import PodsChat from '@/components/pods/PodsChat.vue';
  import MileStoneList from '@/components/pods/MileStoneList.vue';
  import PodsIconsList from '@/components/pods/PodsIconsList.vue';
  import PodRightSideMenu from '@/components/pods/PodRightSideMenu.vue';
  import SubscriptionFeed from '@/components/pods/SubscriptionFeed.vue';
  import Conversation from '@/components/pods/Conversation.vue';
  import MilestoneDetails from '@/components/milestone/MilestoneDetails.vue';
  import ManageMembers from'@/components/modal/ManageMembers.vue';
  import {getMilestonesInfoByUuid} from '../api/Milestone';

  export default {
    name: 'Sample',
        data() {
      return {
        openManageMembers: false,
        openConversation: false,
        openSubscriptionFeed: true,
        openPods: false,
        milestone: null,
      };
    },
    components: {
      PodsChat,
      MileStoneList,
      PodsIconsList,
      PodRightSideMenu,
      SubscriptionFeed,
      Conversation,
      MilestoneDetails,
      ManageMembers,
    },
    async beforeMount() {
      await this.loadMileStoneData();
    },
    watch: {
      '$route.params.id': {
        async handler(val) {
          await this.loadMileStoneData();
        },
        deep: true,
        immediate: true,
      },
    },
    computed: {
      ...mapGetters(['selectedPod']),
    },
    methods: {
      ...mapActions(['getAllPods']),
      close() {
        this.openManageMembers = false;
      },
      openConversationModule() {
        this.openSubscriptionFeed = false;
        this.openConversation = true;
        this.openPods = false;
        this.$router.push({name: 'Pods'});
      },
      openSubscriptionModule() {
        this.openSubscriptionFeed = true;
        this.openConversation = false;
        this.openPods = false;
        this.$router.push({name: 'Pods'});
      },
      openPodsModule() {
        this.openSubscriptionFeed = false;
        this.openConversation = false;
        this.openPods = true;
        this.$router.push({name: 'Pods'});
      },
      async OnChangeMilestone(loadMilestone = true) {
        if (loadMilestone) {
          await this.$store.dispatch('getMilestoneById', this.selectedPod.id);
        }
        await this.loadMileStoneData();
      },
      async loadMileStoneData() {
        if (this.$route.name !== 'Pods' && this.$route.params.id) {
          let resp  = await getMilestonesInfoByUuid(this.$route.params.id);

          this.milestone = resp.data.data[0];
        } else {
          this.milestone = null;
        }
      },
    },
  };
</script>

<style>
  .a,
  .b {
    fill: none;
    stroke: #363853;
    stroke-width: 1.5px;
  }

  .commentIcon {
    fill-rule: evenodd;
  }

  .settingIcon,
  .b {
    fill: none;
    stroke: #afb6bf;
    stroke-width: 1.5px;
  }

  .settingIcon {
    stroke-linecap: round;
    stroke-linejoin: round;
  }

  .imageIcon {
    fill: #afb6bf;
  }

  .sendMessageIcon {
    fill: #38b275;
  }

  .b {
    fill: #fff;
  }
</style>
