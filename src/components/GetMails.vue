<script >
export default {
  

  data() {
    return {
      mails: [],
      isLoading: false,
      url:"",
      itemPerPage:20,
      fromItem:0,
      untilItem:20,
      toItem:0,
      mailsPaginated:[],
      actual_page:1,
      filterDesc:"",
      error: null,
      errorMessage :"",
      subjectMail:"",
      fromMail:"",
      toMail:"",
      dateMail:"",
      contentMail:"",
      totalMailsFound:0,
      isFilter:false,
      VITE_API_ENDPOINT_ALL_MAILS: import.meta.env.VITE_API_ENDPOINT_ALL_MAILS,
      VITE_API_ENDPOINT_FILTER_MAILS : import.meta.env.VITE_API_ENDPOINT_FILTER_MAILS,
      VITE_API_ENDPOINT_TOKEN : import.meta.env.VITE_API_ENDPOINT_TOKEN


    }
  },

  methods: {
    getMails() {

      this.isLoading = true
      if(this.isFilter){
        this.fromItem = 0
        this.actual_page=1
      }
      this.isFilter = false
      this.mails = []
      this.mailsPaginated = []
 
      this.url = this.VITE_API_ENDPOINT_ALL_MAILS
      this.url += '?' + ( new URLSearchParams( {from:this.fromItem,max:this.itemPerPage} ) ).toString();
      console.log('token:'+ this.VITE_API_ENDPOINT_TOKEN)
      fetch(this.url,{
            headers: {
              method: 'get',
              Authorization:'Bearer ' + this.VITE_API_ENDPOINT_TOKEN
              }
          
          })
        .then(response => 
          response.json()
        )
        .then(data => {
         
          this.mails = data.hits.hits
          console.log(data.hits.hits)
          this.totalMailsFound = data.hits.total.value
        })
        .catch(err => {
          this.error = err
          this.errorMessage = err.message;
          
         }
       )
       .then(() => {
        this.isLoading = false
        });
    },
   

    getMailsFilter() {
    
      this.isLoading = true
      if(!this.isFilter){
        this.fromItem = 0
        this.actual_page=1
      }
      this.isFilter = true
      this.mails = []
      this.mailsPaginated = []
      this.url = this.VITE_API_ENDPOINT_FILTER_MAILS
      this.url += '?' + ( new URLSearchParams( {from :this.fromItem,max:this.itemPerPage,filterID:this.filterDesc} ) ).toString();
      console.log('token:'+this.VITE_API_ENDPOINT_TOKEN)
      fetch(this.url,{
              method: 'get',
              headers: {
              Authorization:'Bearer ' + this.VITE_API_ENDPOINT_TOKEN
              }
            })
        .then(response => 
          { if(response.ok) return response.json(); 
            else throw new Error("Status code error :" + response.status) 
          }
        )

        .then(data => {
    
          this.mails = data.hits.hits
          this.totalMailsFound = data.hits.total.value
         
        })
        .catch(err => {
          this.error = err
          this.errorMessage = err.message
     

         }
       )
       .finally(() => {
        this.isLoading = false
      });
    },
    totalPages(){
      return Math.ceil(this.totalMailsFound/this.itemPerPage)
    },

    getPagination(noPages){
      this.actual_page = noPages;


 
    },
    
    getPreviousPage(){
      if(this.actual_page >1){
        this.actual_page --;
        this.fromItem -= this.itemPerPage;

        if(this.totalMailsFound < this.itemPerPage){
          this.untilItem -= this.totalMailsFound;
        }else{
          this.untilItem -= this.itemPerPage;
        }

        if (this.isFilter){
          this.getMailsFilter();
        }else{
          this.getMails();
        }
      }

      this.getPagination(this.actual_page);
     

    },
    getNextPage(){
      if(this.actual_page < this.totalPages()){

        this.actual_page ++;
        this.fromItem += this.itemPerPage;

        if(this.totalMailsFound < this.itemPerPage){

          this.untilItem += this.totalMailsFound;

        }else{

          this.untilItem += this.itemPerPage;
        };

        if (this.isFilter){
          this.getMailsFilter();
        }else{
          this.getMails();
        };

      };

      this.getPagination(this.actual_page);
     
    },
    showMessage(id){

      this.subjectMail =  this.mails[id]._source.Subject;
      this.fromMail =  this.mails[id]._source.From;
      this.toMail = this.mails[id]._source.To;
      this.dateMail = this.mails[id]._source.Date;
      this.contentMail = this.mails[id]._source.Body;
    }
  },
  mounted() {
    this.getMails()
  }
}
</script>

<template>
  <div >
      <!-- component SEARCHING-->
      <div class="flex items-center justify-center p-5">
        <div class="w-full rounded-lg bg-gray-200 p-5 w-2/4">
          <div class="flex">
            <div class="flex w-10 items-center justify-center rounded-tl-lg rounded-bl-lg border-r border-gray-200 bg-white p-5">
              <svg viewBox="0 0 20 20" aria-hidden="true" class="pointer-events-none absolute w-5 fill-gray-500 transition">
                <path d="M16.72 17.78a.75.75 0 1 0 1.06-1.06l-1.06 1.06ZM9 14.5A5.5 5.5 0 0 1 3.5 9H2a7 7 0 0 0 7 7v-1.5ZM3.5 9A5.5 5.5 0 0 1 9 3.5V2a7 7 0 0 0-7 7h1.5ZM9 3.5A5.5 5.5 0 0 1 14.5 9H16a7 7 0 0 0-7-7v1.5Zm3.89 10.45 3.83 3.83 1.06-1.06-3.83-3.83-1.06 1.06ZM14.5 9a5.48 5.48 0 0 1-1.61 3.89l1.06 1.06A6.98 6.98 0 0 0 16 9h-1.5Zm-1.61 3.89A5.48 5.48 0 0 1 9 14.5V16a6.98 6.98 0 0 0 4.95-2.05l-1.06-1.06Z"></path>
              </svg>
            </div>
            <input v-model="filterDesc" type="text" @keyup.enter="getMailsFilter()" class="w-full bg-white pl-2 text-base font-semibold outline-0" placeholder="Search" id="searchID" />
            <input  @click="getMailsFilter()" type="button" value="Buscar" class="bg-blue-500 p-2 rounded-tr-lg rounded-br-lg text-white font-semibold hover:bg-blue-800 transition-colors"/>
          </div>
        </div>
      </div>
  </div>
 
  <div class="relative block md:flex"> 
    <div class="w-full md:w-1/2 relative z-1 bg-gray-100 rounded shadow-lg overflow-hidden">
      <!-- component TABLE -->
        <div class="overflow-hidden rounded-lg border border-gray-200 shadow-md m-5">
          <table class="w-full border-collapse bg-white text-left text-sm text-gray-500">
            <thead class="bg-gray-50">
              <tr>
                <th scope="col" class="px-6 py-4 font-medium text-gray-900">From</th>
                <th scope="col" class="px-0 py-4 font-medium text-gray-900">To</th>
                <th scope="col" class="px-0 py-4 font-medium text-gray-900">Subject</th>
                <th scope="col" class="px-0 py-4 font-medium text-gray-900">Message</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-100 border-t border-gray-100  ">

              <tr  v-if="!isLoading && mails && mails.length" v-for="(item, i) in mails" :key="i"  v-on:click="showMessage(i)" class="hover:bg-gray-50 ">

                      <th class="px-6 py-0  font-normal text-gray-900">
                        <div class="text-sm">
                          <div class="w-48 truncate font-medium text-gray-700">{{ item._source.From }}</div>
                        </div>

                      </th>

                      <th class="px-0 py-0 font-normal text-gray-900">
                        <div class="text-sm">
                          <div class="w-48 truncate font-medium text-gray-700">{{ item._source.To }}</div>
                        </div>
                      </th>

                      <th class="px-0 py-0 font-normal text-gray-900">
                        <div class="text-sm">
                         <td class="max-w-xs truncate px-0 py-1">{{ item._source.Subject }}</td>
                        </div>
                      </th>

                      <td class="p0-6 py-0">
                        <div class="w-32 truncate text-sm">
                          <td class=" p0-6 py-1">{{ item._source.Body }}</td>
                        </div>
                      </td>

              </tr>

              <tr  v-else-if="isLoading">
                <th class="px-6 py-1 font-normal text-gray-900"> Loading...</th>
              </tr>

              <tr  v-else-if="error">
                <th class="px-6 py-1 font-normal text-gray-900"> No Results</th>
              </tr>


            </tbody>

            <tfoot class="bg-gray-50">
              <tr>
                <th scope="row" class="px-6 py-4 font-medium text-gray-900">
                <!-- component PAGINATION -->

                <div class="px-2 flex items-center space-x-4">
                      <span class="text-sm text-gray-500">{{ actual_page }} of {{ totalPages()}} pages </span>
                      <div class="flex items-center space-x-2">
                          <button class="bg-gray-200 text-gray-400 p-1.5 rounded-lg" title="Previous" v-on:click="getPreviousPage()" >
                              <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                  <path fill-rule="evenodd" d="M12.707 5.293a1 1 0 010 1.414L9.414 10l3.293 3.293a1 1 0 01-1.414 1.414l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 0z" clip-rule="evenodd"></path>
                              </svg>
                          </button>
                          <button class="bg-gray-200 hover:bg-gray-300 text-gray-700 p-1.5 rounded-lg transition duration-150" title="Next" v-on:click=" getNextPage()" >
                              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                  <path fill-rule="evenodd" d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z" clip-rule="evenodd"></path>
                              </svg>
                          </button>
                      </div>
                  </div>
                </th >

              </tr>
            </tfoot>

          </table>
        </div>
    </div>

    <div class="w-full md:w-1/2 relative z-1 rounded lg border border-gray-200 overflow-hidden ">
      <div class="overflow-hidden m-5 ">

              <div class="mb-6">
                  <h4 class="text-lg text-gray-800 font-bold pb-2 mb-4 border-b-2">{{ subjectMail }}</h4>
                  <div class="flex items-center justify-between">
                      <div class="flex items-center">
                        
                          <div class="flex flex-col ml-2">
                              <span class="text-sm font-semibold">From : {{fromMail}}</span>
                              <span class="text-xs text-gray-400">To: {{toMail}}</span>
                          </div>
                      </div>
                      <span class="text-sm text-gray-500">{{ dateMail }}</span>
                  </div>
                  <div class="py-6 pl-2 text-gray-700 ">
                      <p>{{contentMail}}</p>
                  </div>
              </div>

      </div>
    </div>

  </div>
 
</template>

