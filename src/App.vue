<template>
    <div class="mb-10">
        <div class="block">
            <div class="border-b border-gray-200">
                <nav class="flex justify-center md:justify-start text-lg" aria-label="Tabs">
                    <a v-for="tab in tabs" :key="tab.name" @click="setCurrent(tab)" :class="[tab.current ? 'border-white' : 'text-slate-500 border-transparent hover:border-white transition duration-200 ', 'whitespace-nowrap py-4 px-5 border-b-2 font-medium text-sm cursor-pointer']" :aria-current="tab.current ? 'page' : undefined">
                        {{ tab.name }}
                    </a>
                </nav>
            </div>
        </div>
        
        <div v-if="tabs[0].current">
            <div class="mt-10">
                <div v-if="loadCD" class="flex flex-wrap w-full justify-center items-center mb-8">
                    <div class="text-center flex items-center font-semibold p-4 rounded">
                        <span class="animate-ping bg-blue-300 w-3 h-3 rounded-full mr-4"/>
                        <p>CHARGEMENT ...</p>
                    </div>
                </div>
                <div class="container mx-auto grid gap-6 grid-cols-1 sm:grid-cols-2 md:grid-cols-3 xl:grid-cols-4">
                    <div class="mx-auto card" v-for="musicVinyl in musicsVinyl" :key="musicVinyl.item">
                        <img v-if="musicVinyl.cover != ''" height="300px" width="300px" :src="musicVinyl.cover" class="w-full hover:scale-105 transition duration-200 cursor-pointer">
                        <img v-else height="300px" width="300px" src="https://res.cloudinary.com/do5ghqhjj/image/upload/v1644873225/error_Cover_108159151f.jpg" class="w-full hover:scale-105 transition duration-200 cursor-pointer">
                        <p class="mt-6 text-base xl:text-sm"><span class="font-bold">{{musicVinyl.name}}</span> - <span class="italic">{{musicVinyl.title}}</span></p>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="tabs[1].current">
            <div class="mt-10">
                <div v-if="loadVinyl" class="flex flex-wrap w-full justify-center items-center mb-8">
                    <div class="text-center flex items-center font-semibold p-4 rounded">
                        <span class="animate-ping bg-blue-300 w-3 h-3 rounded-full mr-4"/>
                        <p>CHARGEMENT ...</p>
                    </div>
                </div>
                <div class="container mx-auto grid gap-6 grid-cols-1 sm:grid-cols-2 md:grid-cols-3 xl:grid-cols-4">
                    <div class="mx-auto card" v-for="musicCD in musicsCD" :key="musicCD.item">
                        <img v-if="musicCD.cover != ''" height="300px" width="300px" :src="musicCD.cover" class="w-full hover:scale-105 transition duration-200 cursor-pointer">
                        <img v-else height="300px" width="300px" src="https://res.cloudinary.com/do5ghqhjj/image/upload/v1644873225/error_Cover_108159151f.jpg" class="w-full hover:scale-105 transition duration-200 cursor-pointer">
                        <p class="mt-6 text-base xl:text-sm"><span class="font-bold">{{musicCD.name}}</span> - <span class="italic">{{musicCD.title}}</span></p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data: () => ({
        tabs: [
            { name: 'Vinyles', current: true },
            { name: 'Albums', current: false },
        ],
        musicsCD: [],
        musicsVinyl: [],
        artistsVinyl: [],
        loadCD: true,
        loadVinyl: true
    }),

    methods: {
        setCurrent(item) {
            let indexCurrent = this.tabs.map(function(e) { return e }).indexOf(item);
            let indexActive = this.tabs.map(function(e) { return e.current }).indexOf(true);

            if (indexCurrent !== indexActive) {
                this.tabs[indexCurrent].current = this.tabs[indexCurrent].current ? false : true;
                this.tabs[indexActive].current = false;
            }
        },
        
        async createdCD() {
            const response = await fetch("https://strapi.martialescudero.com/api/musics?sort[0]=Name:asc&sort[1]=Date:desc&filters[isAlbum][$eq]=true");
            const dataArtists = await response.json();

            let artistsCD = [];

            dataArtists.data.forEach(element => {
                artistsCD.push({
                    name: element.attributes.Name,
                    title: element.attributes.Title
                })
            });

            for (var i = 0; i < artistsCD.length; i++) {
                const response = await fetch("https://ws.audioscrobbler.com/2.0/?method=album.getinfo&api_key=a1b72e5a090a4daea424d6a689ff5711&artist=" + artistsCD[i].name + "&album=" + artistsCD[i].title + "&format=json");
                const dataMusic = await response.json();
                
                var CDname = artistsCD[i].name;
                var CDtitle = artistsCD[i].title;
                var CDcover = dataMusic.album.image[4]["#text"];

                this.musicsCD.push({
                    name: CDname,
                    title: CDtitle,
                    cover: CDcover
                })

                if (i == artistsCD.length - 1) this.loadCD = false;
            }
        },

        async createdVinyl() {
            const response = await fetch("https://strapi.martialescudero.com/api/musics?sort[0]=Name:asc&sort[1]=Date:desc&filters[isVinyl][$eq]=true");
            const dataArtists = await response.json();

            let artistsVinyl = [];

            dataArtists.data.forEach(element => {
                artistsVinyl.push({
                    name: element.attributes.Name,
                    title: element.attributes.Title
                })
            });

            for (var i = 0; i < artistsVinyl.length; i++) {
                const response = await fetch("https://ws.audioscrobbler.com/2.0/?method=album.getinfo&api_key=a1b72e5a090a4daea424d6a689ff5711&artist=" + artistsVinyl[i].name + "&album=" + artistsVinyl[i].title + "&format=json");
                const dataMusic = await response.json();
                
                var Vinylname = artistsVinyl[i].name;
                var Vinyltitle = artistsVinyl[i].title;
                var Vinylcover = dataMusic.album.image[4]["#text"];
                
                this.musicsVinyl.push({ 
                    name: Vinylname,
                    title: Vinyltitle,
                    cover: Vinylcover 
                })
                
                if (i == artistsVinyl.length - 1) this.loadVinyl = false;
            }
        }
    },

    mounted() {
        this.createdCD()
        this.createdVinyl()
    }
}
</script>