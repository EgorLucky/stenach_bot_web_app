<script>
  export default {
    data() {
      return {
        searchText: "",
        isSearching: false,
        isSettingLocation: false,
        settingLocationId: null,
        locationSearchResults: [],
        locationsNotFound: false,
        userLocation: null,
        unauthorized: false,
      }
    },
    async mounted(){
      const accessToken = localStorage.getItem('access_token');
      if(!accessToken)
        await this.getToken()

        await this.getUserLocation();
    },
    methods: {
      async getToken() {
        const data = Telegram.WebApp.initData;

        if (!data) {
          this.unauthorized = true;
        }

        const response = await fetch(`${import.meta.env.VITE_BACKEND_API_URL || ''}/bot/getToken`, {
          method: "post",
          body: data,
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8',
            
          },
        })
         
        if(response.status !== 200) {
          console.log(response)
          return;
        }

        const accessToken = (await response.json()).access_token;
        localStorage.setItem("access_token", accessToken)

      },
      async handleSearchClick() {
        if (this.isSearching)
          return;
        try {
          const textQuery = this.searchText;
          this.isSearching = true;

          const response = await fetch(`${import.meta.env.VITE_BACKEND_API_URL || ''}/weather/searchLocation?query=${textQuery}`, {
            headers: {
              authorization: `telegramWebAppData ${localStorage.getItem('access_token')}`
            }
          });

          if (response.status === 200) {
              this.locationSearchResults = await response.json();
              this.locationsNotFound = !this.locationSearchResults.length
          }
          else {

          }
        }
        catch(error) {
          console.log(error);  
        }

        this.isSearching = false;
      },
      async handleLocationClick(placeId) {
        if (this.isSettingLocation)
          return;
        try {
          this.isSettingLocation = true;
          this.settingLocationId = placeId;

          const response = await fetch(`${import.meta.env.VITE_BACKEND_API_URL || ''}/weather/setLocation?placeId=${placeId}`, {
            method: 'post',
            headers: {
              authorization: `telegramWebAppData ${localStorage.getItem('access_token')}`
            }
          });

          if (response.status === 200) {
              alert('ееееу! локация сохранена!')
              this.userLocation = await response.json();
          }
          else {

          }
        }
        catch(error) {
          console.log(error);  
        }

        this.isSettingLocation = false;
        this.settingLocationId = null;
      },
      async getUserLocation() {
        const response = await fetch(`${import.meta.env.VITE_BACKEND_API_URL || ''}/weather/getLocation`, {
            headers: {
              authorization: `telegramWebAppData ${localStorage.getItem('access_token')}`
            }
          });
        if(response.status === 200) {
          this.userLocation = await response.json();
        }
      }
    }, 
}
</script>

<template>
  <div v-if="!unauthorized">
    <div v-if="userLocation">
      Текущая локация: <b>{{ userLocation.name }}</b>
    </div>
    <div v-else>
      локация не выбрана!
    </div>
    ёёёёёууууу ввади {{!userLocation? "новую": ""}} локацию!
    <br>
    <input
      :disabled="isSearching"
      type="text" 
      v-model="searchText"
      @keyup.enter=handleSearchClick>
    <button 
      :disabled="isSearching"
      @click=handleSearchClick
      style="width: 100%;">
      <span v-if="isSearching" class="loader"></span>
      <span v-else>search</span>
    </button>
    <div>
      <div class="card"
        v-for="location in locationSearchResults" 
        :key="location.placeId"
        @click="() => handleLocationClick(location.placeId)">
        {{ location.displayName }}
        <template v-if="isSettingLocation && settingLocationId === location.placeId">
          <br>
          <span class="loader"></span>
        </template>
      </div>
    </div>
    <div v-if="locationsNotFound">
      Ничего не нашел....
    </div>
  </div>
  <div v-else>
    nnnooot authorized
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #ffffff;
}
</style>
