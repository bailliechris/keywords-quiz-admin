<template>
    <div class="container">

        <!--Log on area-->
        <!--Convert into a separate component!-->
        <!--Token saved to local storage for easier access elsewhere-->
        <div v-if="log_success === false">
            <div class="container">
                <form class="form">
                    <b-field label="Username:">
                        <b-input type="text" value="Username" v-model="user"></b-input>
                    </b-field>
                    <b-field label="Password">
                        <b-input type="password" maxlength="30" v-model="password"></b-input>
                    </b-field>
                    <button type="submit" class="button is-medium is-success" @click="logon_request">
                        Logon
                    </button>
                </form>
            </div>
        </div>

        <!--Log on area end -->

        <div v-if="log_success === true">
            <!--Convert into a separate component!-->
            <!-- Add default value for selection options -->
            <div class="container">
                <b-field label="Filter by Player Name">
                <b-select v-model="selectedName" placeholder="Select a Name" rounded>
                    <option v-for="name in namelist" v-bind:value="name" v-bind:key="name.id">
                    {{ name.name }}
                    </option>
                </b-select>
                </b-field>
                <b-field label="Filter by Topic">
                <b-select v-model="selectedTopic" placeholder="Select a Topic" rounded>
                    <option v-for="topic in topiclist" v-bind:value="topic" v-bind:key="topic.id">
                    {{ topic.name }}
                    </option>
                </b-select>
                </b-field>
<!--
                <div v-if="timelist.length > 0">
                    <b-field label="Filter by Time">
                        <b-select v-model="selectedTime" placeholder="Select a Time" rounded>
                            <option v-for="topic in topiclist" v-bind:value="time" v-bind:key="topic.id">
                            {{ topic.time }}
                            </option>
                        </b-select>
                    </b-field>
                </div>
-->
                <b-button v-on:click="filter_for" class="Button" type="is-info">Filter</b-button>
                <b-button v-on:click="reload_table" class="Button" type="is-warning">Reload Table</b-button>
            </div>

            <!--Show all scores as a table, so the can be sorted-->
            <!--vbind scorelist as the filtered versions-->
            <div class="container">
                <TableOfScores v-bind:scorelist="showlist" />
            </div>

            <div class="container">
                <form class="form">
                    <button type="submit" class="button is-medium is-danger" @click="show_delete">
                        Show for Deleting
                    </button>
                </form>
            </div>

            <div v-if="showdelete === true">
                <!--Show all scores as elements, so the can be deleted-->
                <div v-bind:key="ascore._id" v-for="ascore in scorelist">
                <Score v-bind:ascore="ascore" v-on:del-score="delete_score" />
                </div>
            </div>
    
        </div>
    </div>
</template>

<script>
import Score from './Score'
import TableOfScores from './TableOfScores'
import axios from 'axios'

export default {
    name: 'Admin',
    watch: {
        log_success: function () {
            if (this.log_success === true) {
                this.get_table();
            }
        }
    },
    components: {
        Score,
        TableOfScores
    },
    data(){
        return{
//            timelist: [],
            scorelist: [],
            showlist:[],
            namelist: [],
            topiclist: [],
            selectedTopic:"All",
            selectedName:"All",
 //           selectedTime:"All",
            user: "",
            password: "",
            log_success: false,
            res_msg: "",
            showdelete: false
        }
    },
    methods: {
        reload_table: async function(e) {
            e.preventDefault();
            this.get_table();
        },

        filter_for: async function(e) {
            e.preventDefault();

            let data = {
                "name": this.selectedName.name, 
                "topic": this.selectedTopic.name,
               // "time": this.selectedTime.name
                };

            let jwtToken = localStorage.getItem("quizToken");

            const config = {
                headers: { Authorization: `Bearer ${jwtToken}` }
            };

            await axios.post("http://localhost:5000/api/scores/query", data, config)
                .then((res) => {
                    if(res.data.token === false) {
                        this.log_success = false;
                    }
                    else {
                        this.scorelist = res.data;
                        this.showlist = this.scorelist;
                    }
            });
        },

        show_delete: function (e) {
            e.preventDefault();
            if(this.showdelete){
                this.showdelete = false;
            } else {
                this.showdelete = true;
            }
        },

        delete_score: async function (_id) {
            this.showdelete = false;

            let jwtToken = localStorage.getItem("quizToken");

            const config = {
                headers: { Authorization: `Bearer ${jwtToken}` }
            };

            await axios.delete("http://localhost:5000/api/scores/admin/"+_id, config)
                .then((res) => {
                    if(res.data.token === false) {
                        this.log_success = false;
                    }
                    if(res.status(200)) {
                        this.reload_table();
                    }
                });
        },

        get_table: async function () {

            let jwtToken = localStorage.getItem("quizToken");

            const config = {
                headers: { Authorization: `Bearer ${jwtToken}` }
            };

            await axios.get("http://localhost:5000/api/scores/admin", config)
                .then((res) => {
                    if(res.data.token === false) {
                        this.log_success = false;
                    }
                    else {
                        this.scorelist = res.data;
                        this.showlist = this.scorelist;
                        this.populate_select();
                    }
            });
        },

        logon_request: async function (e) {
            e.preventDefault();
            let data = {
            "user": this.user, 
            "pw": this.password 
            };

            axios.post("http://localhost:5000/api/login", data)
            .then((res) => {
                this.res_msg = res.data.msg;
                if(res.data.token !== false) {
                    this.token = res.data.token;
                    this.log_success = true;
                    localStorage.setItem("quizToken", this.token);
                }
            });
        },

        //Select menu input for filtering - 
        //Name = Functioning
        //Topic = Functioning
        //Time = NOT Functioning - Not enough time stamped entries in the database?
        populate_select: function () {
            //for all elements in scorelist
            this.namelist = [];
            this.topiclist = [];
 //          this.timelist = [];
            this.selectedTopic = "All";
            this.selectedName = "All";
 //           this.selectedTime = "All";

            
            for (let i=0; i<this.scorelist.length; i++){
                if (this.namelist.includes(this.scorelist[i].name) === false) {
                    this.namelist.push(this.scorelist[i].name);
                }
                if (this.topiclist.includes(this.scorelist[i].topic) === false) {
                    this.topiclist.push(this.scorelist[i].topic);
                }
         /*       if (this.timelist.includes(this.scorelist[i].time) === false) {
                    this.timelist.push(this.scorelist[i].time);
                }*/
            }

            let nameindex = 0;
            let buildnamelist = [{id:nameindex, name:"All"}]
        
            for (let i=0; i<this.namelist.length; i++){
                nameindex++;
                buildnamelist.push({id:nameindex, name:this.namelist[i]});
            }

            let topicindex = 0;
            let buildtopiclist = [{id:topicindex, name:"All"}]
        
            for (let i=0; i<this.topiclist.length; i++){
                topicindex++;
                buildtopiclist.push({id:topicindex, name:this.topiclist[i]});
            }
8
 /*          let timeindex = 0;
            let buildtimelist = [{id:timeindex, name:"All"}]
        
            for (let i=0; i<this.timelist.length; i++){
                timeindex++;
                buildtimelist.push({id:timeindex, name:this.timelist[i]});
            } */

            this.topiclist = buildtopiclist;
            this.namelist = buildnamelist;
  //          this.timelist = buildtimelist;
        }
    }
}
</script>