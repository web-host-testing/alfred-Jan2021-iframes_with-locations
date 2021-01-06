<template>
  <div>
    <div class="list-location">
      <TrackingBar @clicked="searchByQuery" :query="query" />
      <div class="dropdown__container">
        <!-- <dropdown label="Any type" :options="types" @choosed="searchByType" /> -->
        <!--Custom Dropdown-->
        <div class="dropdown">
          <button
            class="dropdown__btn"
            :class="{ 'dropdown__btn--active': isOpenType, 'light-blue-dropdown': changedDropdownType}"
            @click="isOpenType = !isOpenType"
          >
            {{ typeSelected ? typeSelected.name : "任何類別" }}
            <svg class="dropdown__icon" v-if="!isOpenType">
              <use
                xlink:href="@/assets/images/sprite.svg#icon-chevron-down"
              ></use>
            </svg>
            <svg class="dropdown__icon" v-else>
              <use
                xlink:href="@/assets/images/sprite.svg#icon-chevron-up"
              ></use>
            </svg>
          </button>
          <button
            v-if="isOpenType"
            tabindex="-1"
            class="dropdown__backdrop"
            @click="isOpenType = !isOpenType"
          ></button>
          <div class="dropdown__content" v-if="isOpenType">
            <span
              class="dropdown__link"
              v-bind:class="{ 'dropdown-not-selected': typeSelected === null}"
              @click="searchByType(null)"
              >任何类型</span
            >
            <div style="display:inline" v-for="(type, keyType) in types" :key="keyType">
              <span
                class="dropdown__link"
                v-bind:class="{ 'light-blue-dropdown': (type && typeSelected && typeSelected.id === type.id)}"
                :style="appliedStyleDropdownCustom(type)"
                @click="searchByType(type)"
              >
                {{ type.name }}
              </span>
              <hr v-if="type.isParent" class="dropdown-border-custom" />
            </div>
          </div>
        </div>
        <!--Custom Dropdown-->
        <div class="dropdown">
          <button
            class="dropdown__btn"
            :class="{ 'dropdown__btn--active': isOpenDistrict, 'light-blue-dropdown': changedDropdownDistrict}"
            @click="isOpenDistrict = !isOpenDistrict"
          >
            {{ districtSelected ? districtSelected.name : "地區" }}
            <svg class="dropdown__icon" v-if="!isOpenDistrict">
              <use
                xlink:href="@/assets/images/sprite.svg#icon-chevron-down"
              ></use>
            </svg>
            <svg class="dropdown__icon" v-else>
              <use
                xlink:href="@/assets/images/sprite.svg#icon-chevron-up"
              ></use>
            </svg>
          </button>
          <button
            v-if="isOpenDistrict"
            tabindex="-1"
            class="dropdown__backdrop"
            @click="isOpenDistrict = !isOpenDistrict"
          ></button>
          <div class="dropdown__content" v-if="isOpenDistrict">
            <span
              class="dropdown__link"
              v-bind:class="{ 'dropdown-not-selected': districtSelected === null}"
              @click="searchByDistrict(null)"
              >地區</span
            >
            <div style="display:inline" v-for="(district, keyDistrict) in districts" :key="keyDistrict">
              <span
                class="dropdown__link"
                v-bind:class="{ 'light-blue-dropdown': (district && districtSelected && districtSelected.id === district.id)}"
                :style="appliedStyleDropdownCustom(district)"
                @click="searchByDistrict(district)"
              >
                {{ district.name }}
              </span>
              <hr v-if="district.isParent" class="dropdown-border-custom" />
            </div>
          </div>
        </div>
        <dropdown
          label="服務類別"
          :options="services"
          @choosed="searchByService"
          :isLastDropdown="true"
        />
      </div>
      <div v-if="locations.length > 0">
        <location-tab
          v-for="(location, index) in locations"
          :key="index"
          v-bind:location="location"
        />
      </div>
      <div
        style="text-align: center; padding-top: 1rem"
        v-if="!isLoading && locations.length < 1"
      >
        <strong>Data not found</strong>
      </div>
      <div style="text-align: center" v-if="isLoading">
        <pulse-loader />
      </div>
      <Pagination
        :paginator="paginator"
        v-if="paginator && locations.length > 0 && !isLoading"
        @changePage="getLocations"
      />
    </div>
    <div class="map">
      <Map :locations="maps" />
    </div>
  </div>
</template>
<script>
import axios from "axios";
import LocationTab from "@/components/LocationTab.vue";
import Map from "@/components/Map.vue";
import Dropdown from "@/components/Dropdown.vue";
import TrackingBar from "@/components/TrackingBar.vue";
import PulseLoader from "vue-spinner/src/PulseLoader.vue";
import Pagination from "@/components/Pagination.vue";

export default {
  name: "LocationList",
  components: {
    LocationTab,
    Map,
    Dropdown,
    TrackingBar,
    PulseLoader,
    Pagination,
  },
  data() {
    return {
      query: null,
      typeSelected: null,
      districtSelected: null,
      serviceSelected: null,
      locations: [],
      maps: [],
      perpage: 20,
      types: [],
      districts: [],
      services: [],
      isOpenType: false,
      isOpenDistrict: false,
      changedDropdownType: false,
      changedDropdownDistrict: false,
      isLoading: false,
      paginator: null,
      redemptionCenter: [],
      smartLocker: [],
    };
  },
  methods: {
    async getTypes() {
      const vm = this;

      vm.types = [];
      vm.redemptionCenter = [];
      vm.smartLocker = [];
      await axios
        .get(`http://159.138.38.21:30081/api/locker/v1/site/findSiteTypeList`, {
          headers: {
            "X-Req-Language": "zh",
          },
        })
        .then(function (response) {
          // handle success
          response.data.forEach(function(type) {
            //Handle Redemption Center
            if (
              type.siteTypeEnmu === 'ALFRED_POINT' ||
              type.siteTypeEnmu === 'FPX_REDEMPTION_CENTER' ||
              type.siteTypeEnmu === 'CIRCLE_K' ||
              type.siteTypeEnmu === 'SEVEN_ELEVEN'
            ) {
              vm.redemptionCenter.push(type);
            }

            //Handle Smart Locker
            if (
              type.siteTypeEnmu === 'FPX_LOCKER' ||
              type.siteTypeEnmu === 'ALFRED_LOCKER'
            ) {
              vm.smartLocker.push(type);
            }
          });

          //Process Final Data Types
          vm.types.push({
            id: 'REDEMPTION_CENTER',
            name: 'Redemption centers',
            isParent: true,
          });
          
          vm.redemptionCenter.forEach(function(type) {
            vm.types.push({
              id: type.siteTypeEnmu,
              name: type.siteTypeName,
              isParent: false,
            });
          });

          vm.types.push({
            id: 'SMART_LOCKER',
            name: 'Smart lockers',
            isParent: true,
          });
          
          vm.smartLocker.forEach(function(type) {
            vm.types.push({
              id: type.siteTypeEnmu,
              name: type.siteTypeName,
              isParent: false,
            });
          });
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        });
    },
    async getDistricts() {
      const vm = this;

      vm.districts = [];
      await axios
        .get(`http://159.138.38.21:30081/api/user/v1/district/listNested`, {
          headers: {
            "X-Req-Language": "zh",
          },
        })
        .then(function (response) {
          // handle success
          response.data.forEach((district) => {
            vm.districts.push({
              id: district.districtId,
              name: district.name,
              isParent: true,
            });
            district.children.forEach((subDistrict) => {
              vm.districts.push({
                id: subDistrict.districtId,
                name: subDistrict.name,
                isParent: false,
              });
            });
          });
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        });
    },
    async getServices() {
      const vm = this;

      vm.services = [];
      await axios
        .get(
          `http://159.138.38.21:30081/api/service/v1/service/all/simpleInfo`,
          {
            headers: {
              "X-Req-Language": "zh",
            },
          }
        )
        .then(function (response) {
          // handle success
          vm.services = response.data.map(function (service) {
            return {
              id: service.lockerServiceName,
              name: service.lockerServiceName,
            };
          });
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        });
    },
    getParamTypes() {
      const vm = this;
      let result = [];

      if (!vm.typeSelected) {
        result = [];
      } else if (vm.typeSelected.id === 'REDEMPTION_CENTER') {
        result.push('ALFRED_POINT');
        result.push('FPX_REDEMPTION_CENTER');
        result.push('CIRCLE_K');
        result.push('SEVEN_ELEVEN');
      } else if (vm.typeSelected.id === 'SMART_LOCKER') {
        result.push('FPX_LOCKER');
        result.push('ALFRED_LOCKER');
      } else if (vm.typeSelected) {
        result.push(vm.typeSelected.id);
      }

      return result;
    },
    async getLocations(page = 0) {
      const vm = this;

      vm.isLoading = true;
      vm.locations = [];
      await axios
        .post(
          `http://159.138.38.21:30081/api/locker/v1/queryForMap`,
          {
            size: vm.perpage,
            page: page,
            queryKeyword: vm.query,
            siteTypes: vm.getParamTypes(),
            level3AriaId: vm.districtSelected ? vm.districtSelected.id : null,
            services: vm.serviceSelected,
          },
          {
            headers: {
              "X-Req-Language": "zh",
              username: "alfredwebsite",
              password: "alfredwebsite",
            },
          }
        )
        .then(function (response) {
          // handle success
          vm.locations = response.data.content;
          vm.paginator = response.data;
          vm.isLoading = false;
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        });
    },
    async getMaps() {
      const vm = this;

      vm.maps = [];
      await axios
        .post(
          `http://159.138.38.21:30081/api/locker/v1/listForMap`,
          {
            queryKeyword: vm.query,
            siteTypes: vm.getParamTypes(),
            level3AriaId: vm.districtSelected ? vm.districtSelected.id : null,
            services: vm.serviceSelected,
          },
          {
            headers: {
              "X-Req-Language": "zh",
            },
          }
        )
        .then(function (response) {
          // handle success
          vm.maps = response.data;
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        });
    },
    handleEscape(e) {
      if (e.key === "Esc" || e.key === "Escape") {
        this.isOpenDistrict = false;
      }
    },
    searchByQuery(query) {
      this.query = query;
      this.getLocations();
      this.getMaps();
    },
    searchByType(type) {
      this.isOpenType = !this.isOpenType;
      this.typeSelected = type;
      this.getLocations();
      this.getMaps();
      if(type === null){
        this.changedDropdownType = false;
      }
      else{
        this.changedDropdownType = true;
      }
    },
    searchByDistrict(district) {
      this.isOpenDistrict = !this.isOpenDistrict;
      this.districtSelected = district;
      this.getLocations();
      this.getMaps();
      if(district === null){
        this.changedDropdownDistrict = false;
      }
      else{
        this.changedDropdownDistrict = true;
      }
    },
    searchByService(service) {
      this.serviceSelected = service;
      this.getLocations();
      this.getMaps();
    },
    appliedStyleDropdownCustom(value) {
      let style = {
        "text-align": "center"
      };
      if (value.isParent) {
        style["font-weight"] = "bold";
        style["font-size"] = "13px";
        style["letter-spacing"] = "-0.3px";
      }

      return style;
    },
  },
  mounted() {
    if (typeof this.$route.query.q !== "undefined") {
      this.query = this.$route.query.q;
    }

    this.getTypes();
    this.getDistricts();
    this.getServices();
    this.getLocations();
    this.getMaps();
  },
  created() {
    document.addEventListener("keydown", this.handleEscape);
  },
  destroyed() {
    document.removeEventListener("keydown", this.handleEscape);
  },
};
</script>