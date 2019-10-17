<template>
  <div id="app">
    <div class="container">

      <div class="row">
        <header class="header">
            <img src="./assets/logo.svg" alt="">
        </header>
      </div>

      <div class="row">

        <div class="filter">

          <p class="filter-title">Количество пересадок</p>

          <input type="radio" name="group" id="stopsAll" v-model="filterPicked"  @change="sortedStops" value="All">
          <label class="filter-item" for="stopsAll">Все</label>

          <input type="radio" name="group" id="stops0" v-model="filterPicked"  @change="sortedStops" value="0">
          <label class="filter-item" for="stops0">Без пересадок</label>
          
          <input type="radio" name="group" id="stops1" v-model="filterPicked"  @change="sortedStops" value="1">
          <label class="filter-item" for="stops1">1 пересадка</label>

          <input type="radio" name="group" id="stops2" v-model="filterPicked"  @change="sortedStops" value="2">
          <label class="filter-item" for="stops2">2 пересадки</label>

          <input type="radio" name="group" id="stops3" v-model="filterPicked"  @change="sortedStops" value="3">
          <label class="filter-item" for="stops3"> 3 пересадки</label>
          
        </div>

        <div class="content">

          <ul class="tabs">
            <li class="tabs-item"  @click='sortedPrice();' v-bind:class="{ 'tabs-item--active': byPrice }">Самый дешевый</li>
            <li class="tabs-item"  @click='sortedTime();' v-bind:class="{ 'tabs-item--active': byTime }">Самый быстрый</li>
          </ul>

          <div class="tickets">

            <template v-for="(ticket, index) in sortedtickets">
              
              <div class="ticket" v-if="index<ticketToShow"  v-bind:key="ticket.price + ticket.segments[0].date">
                
                <div class="ticket-header">

                  <div class="ticket-cost">{{ ticket.price.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, "$1 ")}} {{currentCurrency}}</div>

                  <div class="ticket-logo">
                    <img v-bind:src="'https://pics.avs.io/al_square/28/28/' + ticket.carrier + '@2x.png'" alt="">
                  </div>

                </div>

                <div class="row">
                  <div class="ticket-block">
                    <p class="ticket-block-top">{{ticket.segments[0].origin}} – {{ticket.segments[0].destination}}</p>
                    <p class="ticket-block-bottom">{{ dataString(ticket.segments[0].date, ticket.segments[0].duration) }}</p>
                  </div>
                  <div class="ticket-block">
                    <p class="ticket-block-top">В пути</p>
                    <p class="ticket-block-bottom">{{ toMinutes(ticket.segments[0].duration) }}</p>
                  </div>
                  <div class="ticket-block">
                    <p class="ticket-block-top">{{stopsFunc(ticket.segments[0].stops.length)}}</p>
                    <p class="ticket-block-bottom"> <span v-for="stop in ticket.segments[0].stops" v-bind:key="stop"> {{stop}}</span></p>
                  </div>
                </div>
                <div class="row">
                  <div class="ticket-block">
                    <p class="ticket-block-top">{{ticket.segments[1].origin}} – {{ticket.segments[1].destination}}</p>
                    <p class="ticket-block-bottom">{{ dataString(ticket.segments[1].date, ticket.segments[1].duration) }}</p>
                  </div>
                  <div class="ticket-block">
                    <p class="ticket-block-top">В пути</p>
                    <p class="ticket-block-bottom">{{ toMinutes(ticket.segments[1].duration) }}</p>
                  </div>
                  <div class="ticket-block">
                    <p class="ticket-block-top">{{stopsFunc(ticket.segments[1].stops.length)}}</p>
                    <p class="ticket-block-bottom" > <span v-for="stop in ticket.segments[1].stops" v-bind:key="stop"> {{stop}}</span></p>
                  </div>
                </div>

              </div>

            </template>
            
            <button class="butt-loadmore" @click="ticketToShow += 5">Load more</button>

          </div>

        </div>

      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  data() {
    return {
      tickets: null,
      sortedtickets: null,
      byPrice: true,
      byTime: false,
      filterPicked: 'All',
      currentCurrency: 'rub',
      ticketToShow: 5
    }
  },
  beforeCreate() {
    var vm = this;
    getSearchId()
    function getSearchId(){
      fetch('https://front-test.beta.aviasales.ru/search')
        .then(response => response.json())
        .then(searchId => getTickets(searchId.searchId))
        .catch(err => {getSearchId()});
    }
    function getTickets(searchId){
      fetch(`https://front-test.beta.aviasales.ru/tickets?searchId=${searchId}`)
        .then(response => response.json())
        .then(tickets => {
          vm.tickets = tickets.tickets;
          let byPrice = vm.tickets.slice(0);
          byPrice.sort(function(a,b) {
            return a.price - b.price;
          });
          vm.tickets = byPrice;
          vm.sortedtickets = byPrice;
        })
        .catch(err => {getSearchId()});
    }
    
  }, 
  methods: {
    dataString: function(date, duration){
      // Start
      let dater = new Date(date);
      let startTimeMinutes = (dater.getMinutes() < 10) ? `0${dater.getMinutes()}` : dater.getMinutes();
      let startTimeHours = (dater.getHours() < 10) ? `0${dater.getHours()}` : dater.getHours(); 
      let startTimeString = startTimeHours + ':' +  startTimeMinutes;
      // End
      let endTime = new Date(dater.getTime() + duration*60000);
      let endTimeHourse = (endTime.getHours() < 10) ? `0${endTime.getHours()}` : endTime.getHours(); 
      let endTimeMinutes = (endTime.getMinutes() < 10) ? `0${endTime.getMinutes()}` : endTime.getMinutes();
      let endTimeString = endTimeHourse + ':' +  endTimeMinutes;
      return startTimeString + ' - ' + endTimeString;
    },
    toMinutes: function(sec){
      var time = sec;
      var h = time / 60 ^ 0;
      if (h) {
        var m = time % 60;
        if (m < 10) m = '0' + m;
        time = h + 'ч ' + m + 'м';
      } else {
        time = time + 'м';
      }
      return time;
    },
    stopsFunc: function(stop){
      if(stop == 0){
        return 'Без пересадок';
      }
      else if(stop == 1){
        return '1 пересадка';
      }
      else if(stop >= 1){
        return stop + ' пересадки';
      }
      else if(stop >= 5){
        return stop + ' пересадок';
      }
    },
    sortedPrice: function(){
      let byPrice = this.tickets.slice(0);
      byPrice.sort(function(a,b) {
        return a.price - b.price;
      });
      this.byPrice = true;
      this.byTime = false;
      this.tickets = byPrice;
      this.sortedtickets = byPrice;
    },
    sortedTime: function(){
      let byTime = this.tickets.slice(0);
      byTime.sort(function(a,b) {
        return a.segments[0].duration - b.segments[0].duration;
      });
      this.byPrice = false;
      this.byTime = true;
      this.tickets = byTime;
      this.sortedtickets = byTime;
    },
    sortedStops: function(){
      if(this.filterPicked != 'All'){
        
        let filtredItem = this.filterPicked
        let byStops = this.tickets.slice(0);
        
        let find = byStops.filter(function(result) {
          return result.segments[0].stops.length == filtredItem;
        });

        this.sortedtickets = find;

      }
      else{
        this.sortedtickets = this.tickets;
      }
    }
  },
}
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Open+Sans:300,400,600,700&display=swap');
  *{
    margin: 0;
    padding: 0;
  }
  #app{
    background: #E5E5E5;
    min-height: 100vh;
    font-family: 'Open Sans', sans-serif;
    padding-bottom: 100px;
  }
  .row{
    display: flex;
    width: 100%;
    flex-wrap: wrap;
  }
  .header{
    width: 100%;
    display: flex;
    justify-content: center;
    padding: 50px 0;
  }
  .container{
    max-width: 768px;
    margin: 0 auto;
  }
  .filter{
    position: sticky;
    top: 10px;
    width: calc(30% - 30px);
    height: 100%;
    margin-right: 10px;
    display: flex;
    flex-direction: column;
    padding: 20px;
    background: #FFFFFF;
    box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.1);
    border-radius: 5px; 
  }
  .filter-title{
    text-transform: uppercase;
    color: #4A4A4A;
    font-weight: 600;
    font-size: 12px;
    line-height: 1;
    margin-bottom: 20px;
  }
  .filter-item{
    display: flex;
    align-items: center;
    padding-bottom: 20px;
    font-family: 'Open Sans', sans-serif;
    font-size: 13px;
  }
  .filter-item input{
    margin-right: 6px;
  }
  .filter-item:last-child{
    padding-bottom: 0;
  }
  .content{
    width: calc(70% - 30px);
    margin-left: 10px; 
  }
  .tickets{
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
  .tickets:empty{
    height: 300px;
    width: 100%;
    background: url(./assets/loader.gif) no-repeat;
    background-size: 80px 80px;
    background-position: center center;
  }
  .tabs{
    display: flex;
    padding: 0;
    margin: 0;
    list-style: none;
  }
  .tabs-item{
    width: 50%;
    padding: 15px 0;
    background: #FFFFFF;
    border: 1px solid #DFE5EC;
    text-align: center;
    letter-spacing: 0.5px;
    text-transform: uppercase;
    color: #4A4A4A;
    cursor: pointer;
    transition: .3s all;
  }
  .tabs-currency-item{
    width: 33.33%;
    padding: 8px 0;
    background: #FFFFFF;
    border: 1px solid #DFE5EC;
    text-align: center;
    letter-spacing: 0.5px;
    text-transform: uppercase;
    color: #4A4A4A;
    cursor: pointer;
    transition: .3s all;
  }
  .tabs-currency{
    display: flex;
    padding-bottom: 20px;
  }
  .tabs-item:hover{
    box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.1);
  }
  .tabs-item--active{
    color: #FFFFFF;
    background: #2196F3;
    border: 1px solid #2196F3;
  }
  .ticket{
    background: #FFFFFF;
    padding: 20px;
    box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    margin-top: 20px;
    cursor: pointer;
    transition: .3s all;
  }
  .ticket:hover{
    box-shadow: 0px 6px 12px rgba(0, 0, 0, 0.3);
  }
  .ticket-header{
    display: flex;
    justify-content: space-between;
    align-items: center;
    align-self: auto;
    padding-bottom: 10px;
  }
  .ticket-cost{
    color:#2196F3;
    font-weight: 600;
    font-size: 24px;
  }
  .ticket-block{
    width: 33%;
    padding-top: 10px;
  }
  .ticket-block-top{
    font-weight: 600;
    font-size: 12px;
    text-transform: uppercase;
    color: #A0B0B9;
  }
  .ticket-block-bottom{
    font-weight: 600;
    font-size: 14px;
    line-height: 21px;
    color: #4A4A4A;
  }
  .butt-loadmore{
    width: 255px;
    padding: 10px;
    text-align: center;
    color: #fff;
    background: #2196F3;
    margin: 0 auto;
    margin-top: 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  .butt-loadmore:hover{
    opacity: 0.8;
  }
  label{
    position: relative;
  }
  input[type="radio"]:checked,
  input[type="radio"]:not(:checked) {
    position: absolute;
    left: -9999px;
  }
  input[type="radio"]:checked + label,
  input[type="radio"]:not(:checked) + label
  {
    position: relative;
    padding-left: 28px;
    cursor: pointer;
    line-height: 20px;
    display: inline-block;
    color: #666;
  }
  input[type="radio"]:checked + label:before,
  input[type="radio"]:not(:checked) + label:before {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    width: 18px;
    height: 18px;
    border: 1px solid #ddd;
    border-radius: 100%;
    background: #fff;
  }
  input[type="radio"]:checked + label:after,
  input[type="radio"]:not(:checked) + label:after {
    content: '';
    width: 12px;
    height: 12px;
    background: #2196F3;
    position: absolute;
    top: 4px;
    left: 4px;
    border-radius: 100%;
    -webkit-transition: all 0.2s ease;
    transition: all 0.2s ease;
  }
  input[type="radio"]:not(:checked) + label:after {
    opacity: 0;
    -webkit-transform: scale(0);
    transform: scale(0);
  }
  input[type="radio"]:checked + label:after {
    opacity: 1;
    -webkit-transform: scale(1);
    transform: scale(1);
  }
</style>
