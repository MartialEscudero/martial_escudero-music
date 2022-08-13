<template>
  <div class="home">
    <div class="mb-4 border-b border-gray-200 dark:border-gray-700">
      <ul class="flex flex-wrap -mb-px" id="myTab" data-tabs-toggle="#myTabContent" role="tablist">
        <li class="mr-2" role="presentation">
            <button class="active inline-block py-4 px-4 text-sm font-medium text-center rounded-t-lg hover:text-white" id="dashboard-tab" data-tabs-target="#dashboard" type="button" role="tab" aria-controls="dashboard" aria-selected="true">
              ALBUMS
            </button>
        </li>
        <li class="mr-2" role="presentation">
            <button class="inline-block py-4 px-4 text-sm font-medium text-center rounded-t-lg hover:text-white" id="settings-tab" data-tabs-target="#settings" type="button" role="tab" aria-controls="settings" aria-selected="false">
              VINYLES
            </button>
        </li>
      </ul>
    </div>
    <div id="myTabContent">
      <div class="hidden p-4" id="profile" role="tabpanel" aria-labelledby="profile-tab">
      </div>
      <div class="p-4" id="dashboard" role="tabpanel" aria-labelledby="dashboard-tab">
        <section id="load" class="flex flex-wrap w-full justify-center items-center mb-8">
          <div class="text-center flex items-center text-indigo-50 font-semibold p-4 rounded">
            <span class="animate-ping bg-white w-3 h-3 rounded-full mr-4"> </span>
            CHARGEMENT ...
          </div>
        </section>
        <div class="container mx-auto grid gap-6 grid-cols-1 md:grid-cols-3 xl:grid-cols-4">
          <div class="mx-auto card" v-for="musicCD in musicsCD" :key="musicCD.item">
            <img v-if="musicCD.cover != ''" height="300px" width="300px" :src="musicCD.cover" class="hover:scale-110">
            <img v-else height="300px" width="300px" src="https://res.cloudinary.com/do5ghqhjj/image/upload/v1644873225/error_Cover_108159151f.jpg" class="hover:scale-110">
            <p class="mt-6 text-base xl:text-sm"><span class="font-bold">{{musicCD.name}}</span> - <span class="italic">{{musicCD.title}}</span></p>
          </div>
        </div>
      </div>
      <div class="hidden p-4" id="settings" role="tabpanel" aria-labelledby="settings-tab">
        <div class="container mx-auto grid gap-6 grid-cols-1 md:grid-cols-3 xl:grid-cols-4">
          <div class="mx-auto card" v-for="musicVinyl in musicsVinyl" :key="musicVinyl.item">
            <img v-if="musicVinyl.cover != ''" height="300px" width="300px" :src="musicVinyl.cover" class="hover:scale-110">
            <img v-else height="300px" width="300px" src="https://res.cloudinary.com/do5ghqhjj/image/upload/v1644873225/error_Cover_108159151f.jpg" class="hover:scale-110">
            <p class="mt-6 text-base xl:text-sm"><span class="font-bold">{{musicVinyl.name}}</span> - <span class="italic">{{musicVinyl.title}}</span></p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import 'flowbite';

export default {
  name: 'Home',
  data: () => ({
    musicsCD: [],
    musicsVinyl: [],
    artistsCD : [],
    artistsVinyl : []
  }),
  methods: {
    async createdCD() {
      const response = await fetch("https://strapi.martialescudero.com/api/musics?sort[0]=Name:asc&sort[1]=Date:desc&filters[isAlbum][$eq]=true");
      const dataArtists = await response.json();
      dataArtists.data.forEach(element => {
        this.artistsCD.push(
          {name : element.attributes.Name, title: element.attributes.Title}
        )
      });
      for (var i = 0; i < this.artistsCD.length; i++) {
        const response = await fetch("https://ws.audioscrobbler.com/2.0/?method=album.getinfo&api_key=a1b72e5a090a4daea424d6a689ff5711&artist="+this.artistsCD[i].name+"&album="+this.artistsCD[i].title+"&format=json");
        const dataMusic = await response.json();
        var CDname = this.artistsCD[i].name
        var CDtitle = this.artistsCD[i].title
        var CDcover = dataMusic.album.image[4]["#text"]
        this.musicsCD.push(
          {name : CDname, title: CDtitle, cover: CDcover}
        )
        if (i == this.artistsCD.length-1) {
          document.getElementById("load").classList.add("hidden");
        }
      }
    },
    
    async createdVinyl() {
      const response = await fetch("https://strapi.martialescudero.com/api/musics?sort[0]=Name:asc&sort[1]=Date:desc&filters[isVinyl][$eq]=true");
      const dataArtists = await response.json();
      dataArtists.data.forEach(element => {
        this.artistsVinyl.push(
          {name : element.attributes.Name, title: element.attributes.Title}
        )
      });
      for (var i = 0; i < this.artistsVinyl.length; i++) {
        const response = await fetch("https://ws.audioscrobbler.com/2.0/?method=album.getinfo&api_key=a1b72e5a090a4daea424d6a689ff5711&artist="+this.artistsVinyl[i].name+"&album="+this.artistsVinyl[i].title+"&format=json");
        const dataMusic = await response.json();
        var Vinylname = this.artistsVinyl[i].name
        var Vinyltitle = this.artistsVinyl[i].title
        var Vinylcover = dataMusic.album.image[4]["#text"]
        this.musicsVinyl.push(
          {name : Vinylname, title: Vinyltitle, cover: Vinylcover}
        )
      }
    }
  },
  mounted() {
    this.createdCD()
    this.createdVinyl()
  }
}
</script>

<style>
body {
  font-family: 'Poppins', sans-serif;
  text-align: center;
  color: white;
  background-color: black;
}

p {
  color: white
}

.active {
  border-bottom: 2px solid white;
}
</style>