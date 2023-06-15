<template>
    <div v-if="currentPageContent.length !== 0">
    <table class="table table-bordered table-striped table-hover">
        <thead>
    <tr>
        <th>Select</th>
        <th>Searched Addresses</th>
    </tr>
      </thead>
    <tbody>
        <tr v-for="searchedAddress in currentPageContent" :key="searchedAddress.place_id">
            <td><input type="checkbox" v-model="searchedAddress.selection"></td>
            <td>{{searchedAddress.formatted_address}}</td>
        </tr>
    </tbody>
    </table>
    <ul class="pagination justify-content-center">
        <button class="btn btn-outline-primary mx-3" @click="prevPage">Prev</button>
        <li class="page-item"><a class="page-link" >{{currentPage}}</a></li>
        <button class="btn btn-outline-primary mx-3" @click="nextPage">Next</button>
    </ul>
    </div>
    <div v-if="currentPageContent.length === 0">
        <h2>Currently no search history has been found</h2>
    </div>
</template>

<script>
export default {
    props: ['searchedHistory'], 
    data(){
        return {
            currentPage:1,
            recordsPerPage:10,
            lastPage:1,
            currentPageContent: []
        }
    },
    methods: {
            nextPage: function(){
                if (this.currentPage < this.lastPage){
                    this.currentPage += 1
                    this.pageUpdate()
                } else {
                    alert("this is the last page alreay")
                }
            }, 
            prevPage: function(){
                if (this.currentPage > 1){
                    this.currentPage -= 1
                    this.pageUpdate()
                } else {
                    alert("this is the first page already")
                }
            }, 
            pageUpdate (){
                this.lastPage = Math.ceil(this.searchedHistory.length/this.recordsPerPage)
                this.currentPageContent = this.searchedHistory.slice(this.recordsPerPage * (this.currentPage - 1), this.recordsPerPage * this.currentPage)
            }
    }, 
    mounted(){
        this.pageUpdate()
    }
}
</script>

