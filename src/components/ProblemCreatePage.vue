<template>
    <div class="problem-edit-main">
      <div class="problem-edit-body">
        <div class="edit-div">
          <h3>題目名稱</h3>
          <div class="edit-item"> 
            <input class="title-input" type="text" v-model="title">
          </div>
        </div>
        <div class="edit-div">
          <h3>科目</h3>
          <div class="edit-item">
            <label style="user-select: none;" v-for="option in sujectOptions" :key="option.value" class="radio-option">
            <input
            type="radio"
            :value="option.value"
            v-model="suject"
            name="options"
            />
            {{ option.text }}
            </label>
          </div>
        </div>
        
        <div class="edit-div">
          <h3>題目敘述</h3>
          <div class="edit-item">
            <textarea name="" class="problem-describe" v-model="describe"></textarea>
          </div>
        </div>
        
        <div v-if="this.suject!='program'" style="width: 100%;">
          <div class="edit-div" >
            <h3>題目類型</h3>
            <div class="edit-item">
              <label style="user-select: none;" v-for="option in problemTypeOptions" :key="option.value" class="radio-option">
              <input
              type="radio"
              :value="option.value"
              v-model="problemType"
              name="problemTypeOptions"
              />
              {{ option.text }}
              </label>
            </div>
          </div>
          <div class="edit-div">
            <div class="edit-item">
              <h3 style="margin-top: 16px;">選項(最多五項)</h3>
              <button @click="addAnswerOption" style="margin-left: 10px"> <h3 style="margin: 0;">+</h3></button>
            </div>
            
            <div class="edit-item2">
              <div v-for="(item,idx) in optionList" :key="idx" class="option-items">
              <input type="text" v-model="item.optionName">
              <button style="margin-left: 10px; width: 30px; height: 30px;" @click="removeAnswerOption(idx)"><h3 style="margin: 0;">-</h3></button>
              </div>
            </div>
          </div>
          
          <div class="edit-div">
            <h3>答案</h3>
            <div class="edit-item2" v-if="problemType=='single'">
              <label v-for="(item,idx) in optionList" :key="idx" class="option-items">
              <input 
                type="radio" 
                :value="item.optionName+'_'+idx" 
                v-model="answerForSingle" 
              />
              {{ item.optionName }}
              </label>
            </div>
            <div class="edit-item2" v-if="problemType=='multiple'">
              <label v-for="(item,idx) in optionList" :key="idx" class="option-items">
              <input 
                type="checkbox" 
                :value="item.optionName+'_'+idx" 
                v-model="answerForMutiple" 
              />
              {{ item.optionName }}
              </label>
            </div>
          </div>
        </div>
        
        <div v-if="this.suject=='program'" style="width: 100%;">
          <div class="edit-div">
            <div class="edit-item">
              <h3 style="margin-top: 16px;">標準輸入</h3>
              <button @click="addAnswerOption" style="margin-left: 10px"> <h3 style="margin: 0;">+</h3></button>
            </div>
            <div class="edit-item" v-for="(item, index) in testCaseInput" :key="index">
              <textarea v-model="testCaseInput[index]" class="problem-describe"></textarea>
            </div>
          </div>

          <div class="edit-div">
            <h3>標準輸出</h3>
            <div class="edit-item"  v-for="(item, index) in testCaseOutput" :key="index">
              <textarea  v-model="testCaseOutput[index]" class="problem-describe"></textarea>
              <button style="margin-left: 10px; width: 30px; height: 30px;" @click="removeTestcase(index)"><h3 style="margin: 0;">-</h3></button>
            </div>
          </div>
        </div>
        <button @click="uploadProblem">上傳題目</button>
        <button style="margin-top: 20px;" @click="uploadProblem2">暫存為草稿</button>
      </div>
    </div>
</template>
  
  <script>
  
  export default {
    name: 'ProblemEditPage',
    components:{
      
    },
    data(){
      return {
        publish_status:'',
        itemId:-1,
        title:'',
        suject:'program',
        sujectOptions: [
          { value: 'program', text: '程式' },
          { value: 'math', text: '數學' },
          { value: 'natural', text: '自然' },
        ],
        problemTypeOptions:[
          { value: 'single', text: '單選題' },
          { value: 'multiple', text: '多選題' },
        ],
        problemType:'single',
        describe:'',
        optionList:[],
        answerForSingle:'',
        answerForMutiple:[],
        testCaseInput:[],
        testCaseOutput:[],
      }
    },
    methods:{
      addAnswerOption(){
        if(this.suject!='program' && this.optionList.length<5){
          this.optionList.push({optionName:'新選項 '+String(this.optionList.length+1)});
        }
        else{
          this.testCaseInput.push("新標準輸入");
          this.testCaseOutput.push("新標準輸出");
        }
      },
      removeTestcase(idx){
        this.testCaseInput.splice(idx,1);
        this.testCaseOutput.splice(idx,1);
      },
      removeAnswerOption(idx){
        this.optionList.splice(idx,1);
      },
      uploadProblem(){
        this.publish_status='publish';
        if(this.suject=='natural'){
          this.uploadNaturalData();
        }
        else if(this.suject=='math'){
          this.uploadMathData();
        }
        else{
          this.uploadProgramData();
        }
        this.$router.push({ name: 'MainPage' });
      },
      uploadProblem2(){
        this.publish_status='draft';
        if(this.suject=='natural'){
          this.uploadNaturalData();
        }
        else if(this.suject=='math'){
          this.uploadMathData();
        }
        else{
          this.uploadProgramData();
        }
        this.$router.push({ name: 'MainPage' });
      },
      async uploadNaturalData() {
        try {
          let answer="";
          if(this.problemType=='single'){
            answer=this.answerForSingle;
          }
          else{
            answer=this.answerForMutiple;
          }
          const data={
            'problem_description':this.describe,
            'question_options':this.optionList,
            'answer':answer,
            'problem_type':this.problemType,
          }
          const token=this.access_token;
          let keyword='problem';
          if(this.publish_status=='draft'){
            keyword='draft';
          }
          const response = await fetch(`${this.api_url}/teacher-platform/science-${keyword}-info-list/`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${token}`
            }, 
            body: JSON.stringify(data) 
          });

          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }
          const result = await response.json();
          console.log(result);
        } catch (error) {
          console.error('發送請求時出錯：', error);
        }
      },
      async uploadMathData() {
        try {
          let answer="";
          if(this.problemType=='single'){
            answer=this.answerForSingle;
          }
          else{
            answer=this.answerForMutiple;
          }
          const data={
            'problem_description':this.describe,
            'question_options':this.optionList,
            'answer':answer,
            'problem_type':this.problemType,
          }
          const token=this.access_token;
          let keyword='problem';
          if(this.publish_status=='draft'){
            keyword='draft';
          }
          const response = await fetch(`${this.api_url}/teacher-platform/math-${keyword}-info-list/`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${token}`
            }, 
            body: JSON.stringify(data) 
          });

          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }
          const result = await response.json();
          console.log(result);
        } catch (error) {
          console.error('發送請求時出錯：', error);
        }
      },
      async uploadProgramData() {
        try {
          const data={
            'problem_description':this.describe,
            'title':this.title,
            'testcase_input':this.testCaseInput,
            'testcase_output':this.testCaseOutput,
          }
          const token=this.access_token;
          let keyword='problem';
          if(this.publish_status=='draft'){
            keyword='draft';
          }
          const response = await fetch(`${this.api_url}/teacher-platform/program-${keyword}-info-list/`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${token}`
            }, 
            body: JSON.stringify(data) 
          });

          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }
          const result = await response.json();
          console.log(result);
        } catch (error) {
          console.error('發送請求時出錯：', error);
        }
      },
    },
    computed:{
      
    },
    props: {},
    inject:['access_token','api_url'],
    mounted(){
    }
  }
  </script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
  <style scoped>
  .title-input{
    width: 35%;
  }
  .option-items{
    margin-bottom: 10px;
  }
  .edit-div{
    display: flex;
    flex-direction: column;
    align-items: start;
    width: 100%;
    border: 3px solid rgb(147, 192, 207);
    border-radius: 5px;
    margin: 0px 0px 20px 0px;
    padding: 20px;
  }
  .problem-edit-main{
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .problem-edit-body{
    display: flex;
    flex-direction: column;
    align-items: start;
    width: 65%;
  }
  .edit-item{
    display: flex;
    align-items: center;
    width: 100%;
  }
  .edit-item2{
    display: flex;
    flex-direction: column;
    align-items: start;
    width: 100%;
  }
  .problem-describe{
    width: 100%;
    height: 50vh;
    font-size: 18px;
  }
  input{
    border: 3px solid rgb(146, 198, 215);
    border-radius: 5px;
    padding: 5px;
  }
  label{
    font-weight: bold;
  }
  button{
    font-weight: bold;
    border: 3px solid rgb(146, 198, 215);
    border-radius: 5px;
  }
  h3{
    margin-top: 0px;
  }
  button:hover{
    border-color: rgb(41, 152, 154);
  }
  @media(max-width: 1050px){
    .title-input{
      width: 80%;
    }
  }
  </style>
  