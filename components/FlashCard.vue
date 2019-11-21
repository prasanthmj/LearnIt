<template>
<div class="my-3 p-2" >
<h2>{{set.name}}</h2>

<div v-if="set.questions.length >0 && current < 0" class="my-3">
    <button class="mr-3 btn btn-secondary" @click="back()">Back to Main Page</button>
    <button class="btn btn-primary" @click="start()">Start</button>
</div>
<div v-if="set.questions.length > 0 && current >=0 && state < 3">
<div class="question-box">
    {{cur_q.question}}
</div>
<div class="my-2 pl-3" v-show="state == 2">
    <div class="answer-hint">Answer:</div>
    <div class="answer-display">{{cur_q.answer}} 
        <span v-show="answer_matching" class="ml-2 badge badge-success">&#10003; Correct!</span>
    </div>
</div>
<div class="my-2 pl-3 form-group">
    <input type="text" class="form-control" 
    name="answer" id="answer" v-model="answer" ref="answer_input"
    placeholder="Your answer" @keyup.enter="handleEnter"/>
</div>
<div class="my-4" v-show="state == 2">
    <button class="btn btn-primary" @click="next()">Next &rarr;</button>
</div>
</div>
<div class="my-2" v-show="state==3">
    <h2>Done!</h2>
    <h5 class="mt-2">Sumary</h5>
    <div v-for="(qq,i) in shuffled_questions" :key="i" class="mb-4">
        <div style="font-size:1.1rem;">{{qq.question}}</div>
        <div class="small">{{qq.answer }} <span v-show="qq.matches" class="badge badge-success">&#10003;</span></div>
        <div class="small">{{qq.time }} seconds</div>
    </div>
    <div class="mt-3">
    <button class="mr-3 btn btn-secondary" @click="back()">Back to Main Page</button>

    <button class="btn btn-primary" @click="start()">Do Again</button>
    </div>
</div>
</div>
</template>
<style lang="scss">
.question-box
{
    padding: 2rem 0;
    font-size: 1.5rem;
    color:#333;
}
.answer-hint
{
    font-size: 0.9rem;
    color: #999;
}
.answer-display
{
    font-size:1.2rem;
}

</style>
<script>
import shuffle from "lodash.shuffle";
export default {
    props:{ 
        set:{
                type:Object, 
                default:()=>({questions:[]})
            } 
    },
    data:()=>({ current:-1, state:1, shuffled_questions: [], answer:'', answer_start_time:0 }),
    // state 1 : displaying question
    // 2 : answer entered
    // 3: Finished
    mounted()
    {
        
    },
    computed:
    {
        cur_q()
        {
            if(this.current >=0 && this.current < this.shuffled_questions.length)
            {
                return(this.shuffled_questions[this.current]);
            }
            return null;
        },
        
        answer_matching()
        {
            let right = this.cur_q.answer.toLowerCase().replace(/\s+/g, " ");
            let yours = this.answer.toLowerCase().replace(/\s+/g, " ");
            return(right === yours);
        }
    },
    methods:
    {
        start()
        {
            let qs = shuffle(this.set.questions);
            this.shuffled_questions = qs.map((qust)=>({question:qust.q, 
                answer:qust.a, time:0, 
                matches:false}));

            this.current=0;
            this.initQuestion();
        },
        initQuestion()
        {
            this.state=1;
            this.answer="";
            this.answer_start_time = Date.now();
            this.$nextTick(() => { this.$refs.answer_input.focus(); });
        },
        next()
        {
            if(this.current < this.shuffled_questions.length-1)
            {
                this.current ++;
                this.initQuestion();
            }
            else
            {
                this.answer="";
                this.state =3;
            }
        },
        handleEnter()
        {
            if(this.state == 1)
            {
                let time_now = Date.now();
                this.cur_q.time = Math.round((time_now - this.answer_start_time)/1000 );
                this.cur_q.matches = this.answer_matching;
                this.state=2;
            }
            else if(this.state == 2)
            {
                this.next();
            }
        },
        back()
        {
            this.$emit("done");
        }
    }

}
</script>