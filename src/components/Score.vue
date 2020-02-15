<template>
    <div class = "box">
        <p>
            {{ascore.name}} --- {{ascore.topic}} --- {{ascore.score}} --- {{ascore.max}} --- {{ascore.time}}
            <button v-on:click="$emit('del-score', ascore._id)" class="del">x</button>
        </p>
    </div>
</template>

<script>
export default {
    name: "Score",
    props: ["ascore"]
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