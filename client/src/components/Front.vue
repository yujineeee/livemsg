<template>
    <div class="hello">
        <h1>{{ msg }}</h1>

        <b-container fluid>
            <b-form-textarea id="textarea" style="width:60%; margin-left: auto; margin-right: auto;"
            v-model="postMsg"
            placeholder="Type your question"
            :rows="4"
            :max-rows="4">
        </b-form-textarea><br>
        <!--
        <b-card title="Ask the speaker"
        img-src="https://placekitten.com/1000/300"
        img-alt="Image"
        img-top
        tag="article"
        style="max-width: 40rem;"
        class="mb-2">
        <b-form-textarea no-resize id="textarea" style="width:30rem; margin-left: auto; margin-right: auto;"
        v-model="postMsg"
        placeholder="Type your question"
        :rows="4"
        :max-rows="4">
    </b-form-textarea><br>
-->
        <b-button variant="success" @click="submit()">ASK</b-button>
        </b-card>

        <hr>
		<b-row>
        <div class="div_2" style="text-align: left">{{ history.length }} question</div>
		<div class="div_2" style="text-align: right"><b-form-select style="width:170px" v-model="selected" :options="options" @change="sortFunc" class="mb-3"/></div>
		</b-row>
        <div v-for="key in history">
            <div class="card">
                <div class="container">
                    <b-row>
                        <b-col style="flex: 0 0 90%;max-width: 90%; color: rgba(0,0,0,.4);">{{ key.timestamp| moment("MMMM DD, YYYY HH:mm:ss") }}</b-col>
                    </b-row>
                    <b-row>
                        <b-col style="flex: 0 0 90%;max-width: 90%; color: rgba(0,0,0,.4);">{{ key.postMsg }}</b-col>
                    </b-row>
                    <b-row>
                        <b-button size="sm" style="margin-left: 10px" @click="upvote(key)">{{ key.score }} upvote</b-button>
                    </b-row>
                </div>
            </div>
        </div>
        </b-container>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    name: 'HelloWorld',
    data ()
    {
        return {
            msg		: 'Welcome to Q&A',
            postMsg	: '',
            history	: [ ],
			selected: 'Popular questions',
			options: [ 'Popular questions', 'Recent questions' ]
        }
    },
    mounted()
    {
        this.getTimeline();
        this.todo();
    },
    methods:
    {
		sortFunc()
		{
			this.$nextTick(function()
			{
				if ( this.selected == "Recent questions" )
				{
					this.history.sort(function (a, b) {
						if (a.timestamp > b.timestamp) {
							return -1;
						}
						if (a.timestamp < b.timestamp) {
							return 1;
						}
						return 0;
					});
				}
				else if ( this.selected == "Popular questions" )
				{
					this.history.sort(function (a, b) {
						if (a.score > b.score) {
							return -1;
						}
						if (a.score < b.score) {
							return 1;
						}
						return 0;
					});
				}
			});
		},
        upvote(post)
        {
            var msg = JSON.stringify({'timestamp': post.timestamp, 'msg': post.postMsg});
            axios.post('http://localhost:7005/upvote', {
                key : msg
            })
            .then(response => {
                this.getTimeline();
            })
            .catch(function (error) {
                console.log(error);
            });
        },
        todo()
        {
            setInterval(() => {
                this.getTimeline();
            }, 3000);
        },
        submit()
        {
            let timestamp = Date();
            axios.post('http://localhost:7005/writePost', {
                timestamp : timestamp,
                msg : this.postMsg
            })
            .then(response => {
                console.log(response);
                this.postMsg = '';
                this.getTimeline();
            })
            .catch(function (error) {
                console.log(error);
            });
        },
        getTimeline()
        {
            axios.get('http://localhost:7005/timeline')
            .then(response => {
                this.history = [];
                for (var i = 0, j = 0; i < response.data.length; i+=2, j++)
                {
                    this.history[j] = [];
                    //this.history[response.data[i]] = response.data[i+1];
                    this.history[j]['timestamp'] = JSON.parse(response.data[i]).timestamp;
                    this.history[j]['postMsg'] = JSON.parse(response.data[i]).msg;
                    this.history[j]['score'] = response.data[i+1];
                }
				this.sortFunc();
            })
            .catch(function (error) {
                console.log(error);
            });
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
    font-weight: normal;
    color: rgba(0,0,0,.4);
}
ul {
    list-style-type: none;
    padding: 0;
}
li {
    display: inline-block;
    margin: 0 10px;
}
a {
    color: #42b983;
}
.card {
    box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
    transition: 0.3s;
    width: 60%;
    border-radius: 5px;
    margin-left: auto;
    margin-right: auto;
    text-align: left;
}

.card:hover {
    box-shadow: 0 8px 16px 0 rgba(0,0,0,0.2);
}

img {
    border-radius: 5px 5px 0 0;
}

.container {
    padding: 16px 20px;
}

.div_2 {
    transition: 0.3s;
    border-radius: 5px;
    margin-left: auto;
    margin-right: auto;
	color: rgba(0,0,0,.4);
}
</style>
