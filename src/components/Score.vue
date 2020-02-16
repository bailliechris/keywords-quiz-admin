<template>
    <div class = "box">
        <p>
            {{ascore.name}} --- {{ascore.topic}} --- {{ascore.score}} --- {{ascore.max}} --- {{ascore.time}}
            <button v-on:click="delete_score" class="del">x</button>
        </p>
    </div>
</template>

<script>
import axios from 'axios'

//$emit('del-score', ascore._id)

export default {
    name: "Score",
    props: ["ascore"],
    methods: {
        delete_score: async function () {

            let jwtToken = localStorage.getItem("quizToken");

            const config = {
                headers: { Authorization: `Bearer ${jwtToken}` }
            };

            await axios.delete("https://quiz-scores-admin.herokuapp.com/api/scores/admin/"+this.ascore._id, config)
                .then((res) => {
                    if(res.data.token === false) {
                        //emit to Admin
                        this.$emit('del-score', false);
                    } else {
                        this.$emit('del-score', true);
                    }
                });
        }   

    }
}


</script>

<style scoped>
.todo-item{
    background:cornsilk;
    padding: 10px;
    border-bottom: 1px #ccc dotted;
}

.is-complete {
    text-decoration:line-through;
}

.del {
    background: red;
    color:white;
    border:none;
    padding: 5px 9px;
    border-radius: 50%;
    cursor: pointer;
    float: right;
}
</style>