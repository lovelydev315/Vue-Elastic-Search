<template>
  <div id="app" class="container">
    <div style="position:relative;margin-top:100px" tabIndex="1" v-click-outside="handleBlur">
      <div class="input-group input-group-lg"  @click="drop_show=true">
        <div class="input-group-prepend">
          <span class="input-group-text">Search</span>
        </div>
        <input type="text" 
          class="form-control" 
          v-model="query" 
          @keyup.prevent="search"
          />
      </div>
      <div v-if="drop_show"  class="dropdown-container" :v-show="drop_show">
        <div   v-for="(value, index) in data" 
          :key="index"
          :ref="`card_${index}`"
          class="question-container"
        >
          <h3 class="question-text" @click="questionClick(value)" v-html="highlight(value._source.question)">{{value._source.question}}</h3>
        </div>
      </div>
    </div>
    <div class="text-box" ref="answer_text">
      <h2>{{selected.question}}</h2>
      <p>{{selected.answer}}</p>
    </div>
  </div>
</template>
<script>

import ClickOutside from 'vue-click-outside';

export default {
  data() {
    return {
      query: '',
      drop_show:false,
      selected: { question: "", answer: "" },
      data: [
        
      ]
    }
  },
  directives: {
    ClickOutside
  },
  methods: {
    handleBlur () {
      this.drop_show=false;
    },
    questionClick(item){
      this.drop_show=false;
      this.selected.question = item._source.question;
      this.query = item._source.question;
      this.selected.answer = item._source.answer;
    },
    highlight(question) {
      return question.replace(this.query, '<span style="color:red">'+this.query+'</span>');
    },
    search() {
      this.drop_show=true;
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");

      var raw = JSON.stringify({"query":{"match_phrase":{"question":this.query}}});

      var requestOptions = {
        method: 'POST',
        headers: myHeaders,
        body: raw,
        redirect: 'follow'
      };

      fetch("https://search-streaminghub2-bs2uras4whg2cwjoyif4ybbnle.ap-south-1.es.amazonaws.com/test_index_3/_search", requestOptions)
        .then(response => response.text())
        .then(result => {
          this.data = JSON.parse(result).hits.hits;
          });
    }
  }
}
</script>

<style>
body {
  background-color: #E1E7E7;
}

.dropdown-container {
  background-color:white;
  position:absolute;
  width:100%;
  margin-top:5px;
  border-radius: 5px;
  border: 1px solid gray;
  padding-top: 10px;
  padding-bottom: 10px;
}

.question-text{
  padding-left:20px;
  text-overflow:ellipsis;
  word-wrap: nowrap;
  overflow: hidden;
}

.question-container {
  border-left: 2px solid white;
}

.question-container:hover {
  border-left: 2px solid #ff4901;
  background-color:#b4b4b4;
  color:white;
  cursor:pointer;
}

.text-box {
    margin-top: 20px;
    border: 1px solid gray;
    border-radius: 5px;
    padding: 20px;
    font-size: 25px;
    background-color:aliceblue;
}
</style>

