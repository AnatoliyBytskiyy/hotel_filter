<template>
  <b-container>
    <b-row>
      <b-col cols="4">
        <b-row class="my-2">
          <b-col>
            <label for="search">Страна</label><br>
            <div class="search">
              <b-icon icon="search"></b-icon>
              <input id="search" type="search" placeholder="Поиск стран" v-model="search">
            </div>
          </b-col>
        </b-row>
        <b-row class="my-2">
          <b-col>
            <div class="border country_list">
              <div v-if="countries.length == 0" class="no_countries">
                К сожалению, по вашему запросу ничего не найдено :(
              </div>
              <b-form-checkbox-group
                  v-model="selected_country"
                  :options="countries"
                  value-field="name"
                  text-field="name">
              </b-form-checkbox-group>
            </div>
          </b-col>
        </b-row>
        <b-row class="my-2">
          <b-col>
            <label>Тип</label><br>
            <div class="type border" style="background-color: #FAFAFA;">
              <b-form-checkbox value="Апартаменты" v-model="apartments">Апартаменты</b-form-checkbox>
              <hr>
              <b-form-checkbox value="Отель" v-model="hotel">Отель</b-form-checkbox>
            </div>
          </b-col>
        </b-row>
        <b-row class="my-2">
          <b-col>
            <label>Количество звезд</label><br>
            <div class="type border">
              <b-form-checkbox-group v-model="stars">
                <b-form-checkbox value="1">1 звезда</b-form-checkbox>
                <b-form-checkbox value="2">2 звезды</b-form-checkbox>
                <b-form-checkbox value="3">3 звезды</b-form-checkbox>
                <b-form-checkbox value="4">4 звезды</b-form-checkbox>
                <b-form-checkbox value="5">5 звезд</b-form-checkbox>
              </b-form-checkbox-group>
            </div>
          </b-col>
        </b-row>
        <b-row class="my-2">
          <b-col>
            <label>Количество отзывов (от)</label><br>
            <b-form-input type="text" placeholder="Например, от 10" v-model="reviews" @keypress="isNumber($event)"></b-form-input>
          </b-col>
        </b-row>
        <b-row class="my-2">
          <b-col>
            <label>Цена</label><br>
            <div class="price">
              <b-form-input type="number" nim="0" max="100500" class="price_begin" readonly placeholder="от 0 Р" v-model="price_begin"></b-form-input> - <b-form-input type="number" nim="0" max="100500" class="price_end" readonly placeholder="до 100 500 Р" v-model="price_end"></b-form-input>
            </div>
            <VueSimpleRangeSlider
                :min="0"
                :max="100500"
                :logarithmic="true"
                v-model="range"
            />
          </b-col>
        </b-row>
        <b-row class="my-5">
          <b-col>
            <b-button id="subForm" v-on:click="filter">Применить фильтр</b-button>
            <b-button class="clearForm" v-on:click="clear"><b-icon icon="x"></b-icon>Очистить фильтр</b-button>
          </b-col>
        </b-row>
      </b-col>
      <b-col cols="8">
        <b-row>
          <b-col cols="12" class="my-1" :key="index" v-for="(hotel, index) in paginatedItems">
            <b-card>
              <div class="header">
                <div class="header_info">
                  <p class="info_title">{{ hotel.name }}</p>
                  <div class="parametrs">
                    <div class="parametrs_stars">
                      <b-form-rating id="rating"
                                     v-model=hotel.stars
                                     no-border size="sm"
                                     color="#ff8800"
                                     readonly
                      >
                      </b-form-rating>
                    </div>
                    <div class="parametrs_type">
                      {{ hotel.type }} &bull;
                    </div>
                    <div class="parametrs_reviews">
                      {{ hotel.reviews_amount }} отзыва
                    </div>
                    <div class="parametrs_country">
                      <b-icon icon="geo-alt" class="ico_txt"></b-icon> {{ hotel.country }}
                    </div>
                  </div>
                </div>
                <div class="header_price">
                  <div class="price_number">{{ hotel.min_price }} &#8381;</div>
                  <div class="price_text">Цена за 1 ночь</div>
                </div>
              </div>
              <div class="description">
                <p>{{ hotel.description }}</p>
                <b-button v-bind:id="'btn_check_' + index" class="btn_check"><b-icon icon="calendar4" class="ico_txt"></b-icon>Забронировать</b-button>
              </div>
            </b-card>
          </b-col>
        </b-row>
        <b-row class="pagination" v-if="totalRows != 0">
          <b-col md="6" class="my-1">
            <b-pagination
                @change="onPageChanged"
                :total-rows="totalRows"
                :per-page="perPage"
                v-model="currentPage"
                class="my-0"
                first-number
                last-number
            />
          </b-col>
        </b-row>
        <b-row class="pagination" v-else>
          <b-col class="my-1">
            <b-card>
              <div class="no_countries py-5">
                <img src="../assets/nothing_found.png" alt="Записей не найдено" class="pb-4">
                <label>По данным параметрам ничего не найдено</label>
                <p>Попробуйте изменить параметры фильтрации или вернуться в общий каталог</p>
                <b-button class="btn_check pt-2" v-on:click="clear">Очистить фильтр</b-button>
              </div>
            </b-card>
          </b-col>
        </b-row>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import VueSimpleRangeSlider from 'vue-simple-range-slider';
import 'vue-simple-range-slider/dist/vueSimpleRangeSlider.css'
import json from '../assets/hotels.json';
import countries from '../assets/countries.json';

export default {
  components: { VueSimpleRangeSlider },
  name: 'HotelView',
  props: {
    msg: String
  },
  data() {
    return {
      range: [0,100500],
      hotels: json.hotels,
      search: '',
      countries: countries.countries,
      selected_country: [],
      apartments: '',
      hotel: '',
      stars: [],
      reviews: '',
      price_begin: 0,
      price_end: 100500,
      perPage: 3,
      currentPage: 1,
      paginatedItems: json.hotels,
      totalRows: json.hotels.length
    }
  },
  methods: {
    clear: function() {
      this.selected_country = [];
      this.apartments = false;
      this.hotel = false;
      this.stars = [];
      this.reviews = 0;
      this.range[0] = 0;
      this.range[1] = 100500;
      this.price_begin = 0;
      this.price_end = 100500;
      this.search = '';
      this.paginatedItems = json.hotels;
      this.hotels = json.hotels;

      this.currentPage = 1;
      this.totalRows = this.paginatedItems.length;
      this.paginate(this.perPage, 0);
    },
    filter: function() {
      var selected_country = this.selected_country;
      var apartments       = this.apartments;
      var hotel            = this.hotel;
      var stars            = this.stars;
      var reviews          = this.reviews;
      var price_begin      = this.price_begin;
      var price_end        = this.price_end;

      this.hotels          = json.hotels;
      this.hotels = this.hotels.filter(function (elem) {
        if(selected_country.length == 0) return true;
        else {
          for(let i=0; i<selected_country.length; i++) {
            if(elem.country.indexOf(selected_country[i]) > -1) {
              return elem.country;
            }
          }
        }
      });

      this.hotels = this.hotels.filter(function (elem) {
        if (apartments == '' || (apartments == 'Апартаменты' && hotel == 'Отель')) return true;
        else return elem.type.indexOf(apartments) > -1;
      });

      this.hotels = this.hotels.filter(function (elem) {
        if (hotel == '' || (apartments == 'Апартаменты' && hotel == 'Отель')) return true;
        else return elem.type.indexOf(hotel) > -1;
      });

      this.hotels = this.hotels.filter(function (elem) {
        if (stars.length == 0) return true;
        else {
          for(let i=0; i<stars.length; i++) {
            if(elem.stars == stars[i]) {
              return elem.stars;
            }
          }
        }
      });

      this.hotels = this.hotels.filter(function (elem) {
        if (reviews == 0) return true;
        else {
          if(elem.reviews_amount >= reviews) {
            return elem.reviews_amount;
          }
        }
      });

      this.hotels = this.hotels.filter(function (elem) {
        if (price_begin == 0) return true;
        else {
          if(elem.min_price >= price_begin) {
            return elem.min_price;
          }
        }
      });

      this.hotels = this.hotels.filter(function (elem) {
        if (price_end == 0) return true;
        else {
          if(elem.min_price <= price_end) {
            return elem.min_price;
          }
        }
      });

      this.currentPage = 1;
      this.totalRows = this.hotels.length;
      this.paginate(this.perPage, 0);
    },
    isNumber: function(evt) {
      evt = (evt) ? evt : window.event;
      var charCode = (evt.which) ? evt.which : evt.keyCode;
      if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
        evt.preventDefault();
      } else {
        return true;
      }
    },
    paginate(page_size, page_number) {
      let itemsToParse = this.hotels;
      this.paginatedItems = itemsToParse.slice(
          page_number * page_size,
          (page_number + 1) * page_size
      );
    },
    onPageChanged(page) {
      this.paginate(this.perPage, page - 1);
    }
  },
  mounted() {
    this.paginate(this.perPage, 0);
  },
  watch: {
    search: function () {
      var search     = this.search;
      this.countries = countries.countries
      this.countries = this.countries.filter(function (elem) {
        if(search === '') return true;
        else return elem.name.indexOf(search) > -1;
      })
    },
    range: function (val) {
      this.price_begin = val[0];
      this.price_end   = val[1];
      return;
    }
  }
}
</script>

<style scoped lang="scss">
.content {
  width: 60%;
}

label {
  color: black;
  font-weight: bold;
}

.search {
  display: flex;
  align-items: center;
  border: 1px solid #ced4da;
  line-height: 1.5;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  font-weight: 400;
  svg {
    margin-right: 10px;
  }
  input {
    width: 100%;
    border: none;
    &:focus {
      border: none;
      outline: none;
    }
  }
}

.btn_check {
  display: flex;
  justify-content: center;
  width: 180px;
  height: 40px;
  background-color: rgba(106, 83, 245, 0.5);
  color: #6A53F5;
  &:focus, &:hover, &:active {
    background-color: rgba(106, 83, 245, 0.5) !important;
  }
}

.border {
  padding: 0.375rem 0.75rem;
  margin-bottom: 10px;
}

hr {
  margin-top: 0.3rem;
  margin-bottom: 0.3rem;
}

.price {
  display: flex;
  &_begin {
    margin-right: 10px;
  }
  &_end {
    margin-left: 10px;
  }
  &_number {
    font-weight: bold;
    font-size: 20px;
  }
}

#range {
  margin-top: 10px;
}

#subForm {
  background-color: #6A53F5;
}

.clearForm {
  background-color: inherit;
  color: black;
  font-weight: bold;
}

#subForm,
.clearForm {
  width: 100%;
}

#subForm + .clearForm {
  margin-top: 10px;
}

.header {
  display: flex;
  justify-content: space-between;
}

.info_title {
  color: black;
  text-align: left;
  margin-bottom: 0px;
}

.parametrs {
  display: flex;
  div {
    margin-right: 10px;
    display: flex;
    align-items: center;
  }
}

#rating {
  padding: 0px;
}

.ico_txt {
  margin-right: 5px;
}

.description {
  display: flex;
  align-items: center;
  padding-top: 10px;
  p {
    text-align: left;
  }
}

.search,
.border,
#subForm,
.clearForm {
  border-radius: 10px;
}

.country_list {
  height: 200px;
  overflow-y: auto;
  overflow-x: hidden;
}

.custom-control-inline {
  display: block !important;
}

.pagination {
  justify-content: center;
}

.no_countries {
  text-align: center;
  width: 50%;
  margin: 0px auto;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  height: 180px;
  img {
    width: 200px;
  }
}

.card-body {
  .no_countries {
    height: auto !important;
  }
}
</style>
