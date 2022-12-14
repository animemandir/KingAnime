<script>
	import { goto } from '$app/navigation';
	import { THEME } from '$lib/stores';
	import { onMount } from 'svelte';

    /**
	 * @type {number}
	 */
    let currentTheme;
    let slide = 0;
    /**
	 * @type {number}
	 */
    let slidesAmount;
    /**
	 * @type {string | number | NodeJS.Timeout | undefined}
	 */
    let timer;
    let loaded = false;

    onMount(() => {
        setTimeout(autoSlide, 5000);
    })

    THEME.subscribe(value => {
        currentTheme = value;
    }) 

    function nextSlide() {
        if(slide + 1 >= slidesAmount) {
            slide = 0;
        }
        else slide++;
        moveSlide(slide);
    }
    function lastSlide() {
        if((slide - 1 )< 0) {
            slide = slidesAmount - 1;
        }
        else slide--;
        moveSlide(slide);
    }
    /**
	 * @param {number} i
	 */
    function moveSlide(i) {
        let pos = i + 1;
        // @ts-ignore
        document.getElementById("carousel").style.transform = "translateX(-" + pos + "00%) perspective(1px)";
        autoSlide();
    }
    function autoSlide() {
        clearTimeout(timer);
        timer = setTimeout(() => nextSlide(), 2000);
    }
    /**
	 * @param {{ bannerImage: null; coverImage: any; }} anime
	 */
    function fixBanner(anime) {
        if(anime.bannerImage == null) {
            return new Promise((resolve) => {
                resolve(anime.coverImage);
            })
        }
        else {
            return new Promise((resolve) => {
                resolve(anime.bannerImage);
            })
        }
    }
    /**
	 * @param {{ english: null; userPreferred: any; }} title
	 */
     function fixAnimeTitle(title) {
        if(title.english == null) {
            return new Promise((resolve) => {
                resolve(title.userPreferred);
            })
        }
        else {
            return new Promise((resolve) => {
                resolve(title.english);
            })
        }
    }
    /**
	 * @param {any} animes
	 */
    async function fixData(animes) {
        for (const anime of animes) {
            anime.bannerImage = await fixBanner(anime);
            const newAnimeTitle = await fixAnimeTitle(anime.title);
            anime.title = newAnimeTitle;
        }
        return animes;
    }
    async function fetchPopular() {
        console.log("Fetching Popular Animes");
        const response = await fetch('https://api.enime.moe/popular');
        const result = await response.json(); 
        slidesAmount = result.data.length;
        const animes = fixData(await result.data); 
        loaded = true;
        return animes;
    }
    /**
	 * @param {string | URL} path
	 */
    function transitionStart(path) {
        // @ts-ignore
        document.getElementById('transition-screen').style.opacity = 1;
        setTimeout(() => {
            goto(path);
        }, 1500);
    }
</script> 

<main>
    <!-- svelte-ignore empty-block -->
    {#await fetchPopular()}
    {:then popular_animes}
    <div class="next-btn" class:gold={currentTheme == 0} class:crimson={currentTheme == 1} on:click={() => nextSlide()}>
        <span class="material-symbols-outlined">
        arrow_forward_ios
        </span>
    </div>
    <div class="last-btn" class:gold={currentTheme == 0} class:crimson={currentTheme == 1} on:click={() => lastSlide()}>
        <span class="material-symbols-outlined">
        arrow_back_ios
        </span>
    </div>
    <div id="carousel" class:back-light={currentTheme == 1}> 
        <div class="slide" style={
            "background: linear-gradient(rgba(36, 36, 36, 0), rgba(36, 36, 36, 0),rgba(36, 36, 36, 0.4), rgba(36, 36, 36, 0.7)), url('" + popular_animes[popular_animes.length - 1].bannerImage + "');" +
            "background-size: cover;" + 
            "background-position: center;"            
            }>
            <h3 class:gold={currentTheme == 0} class:crimson={currentTheme == 1}>
                {#each popular_animes[popular_animes.length - 1].genre as genre}
                    <span>{genre}</span>
                {/each}
            </h3>
            <h2>{popular_animes[popular_animes.length - 1].title}</h2>
        </div>
        {#each popular_animes as anime, i}
        <div on:click={() => transitionStart('/anime/' + anime.slug)} class="slide" class:active={slide == i} style={
            "background: linear-gradient(rgba(36, 36, 36, 0), rgba(36, 36, 36, 0),rgba(36, 36, 36, 0.4), rgba(36, 36, 36, 0.7)), url('" + anime.bannerImage + "');" +
            "background-size: cover;" + 
            "background-position: center;"            
            }>
            <h3 class:gold={currentTheme == 0} class:crimson={currentTheme == 1}>
                {#each anime.genre as genre}
                    <span>{genre}</span>
                {/each}
            </h3>
            <h2>{anime.title}</h2>
        </div>
        {/each}
        <div class="slide" style={
            "background: linear-gradient(rgba(36, 36, 36, 0), rgba(36, 36, 36, 0),rgba(36, 36, 36, 0.4), rgba(36, 36, 36, 0.7)), url('" + popular_animes[0].bannerImage + "');" +
            "background-size: cover;" + 
            "background-position: center;"            
            }>
            <h3 class:gold={currentTheme == 0} class:crimson={currentTheme == 1}>
                {#each popular_animes[0].genre as genre}
                    <span>{genre}</span>
                {/each}
            </h3>
            <h2>{popular_animes[0].title}</h2>
        </div>
    </div>
    {/await}
</main>

<style lang="scss">
    main {
        width: 100vw;
        height: 45vh;
        overflow: hidden;
        display: grid;
        place-items: center;
        position: relative;
        background: linear-gradient(rgba(36, 36, 36, 0), rgba(36, 36, 36, 0.1), rgba(36, 36, 36, 0.1));
    }
    #carousel {
        width: 30vw;
        display: flex;
        flex-wrap: nowrap;
        align-items: center;
        transition: 0.25s ease-in-out;
        margin-left: -10.5vw;
        transform: translateX(-100%);
        opacity: 0;
        animation: fadeIn;
        animation-duration: 1.25s;
        animation-timing-function: ease-in-out;
        animation-fill-mode: forwards;
    }
    .next-btn {
        position: absolute;
        padding: 1rem;
        right: 20%;
        color: white;
        z-index: 1;
        transform: scale(2);
        cursor: pointer;
        user-select: none;
        transition: 0.25s ease-in-out;
        opacity: 0;
        animation: fadeIn;
        animation-duration: 1.25s;
        animation-timing-function: ease-in-out;
        animation-fill-mode: forwards;
        
        &:active {
            transform: scale(3);
            color: white;
        }
    }
    .last-btn {
        position: absolute;
        padding: 1rem;
        left: 20%;
        color: white;
        z-index: 1;
        transform: scale(2);
        cursor: pointer;
        user-select: none;
        transition: 0.25s ease-in-out;
        opacity: 0;
        animation: fadeIn;
        animation-duration: 1.25s;
        animation-timing-function: ease-in-out;
        animation-fill-mode: forwards;

        &:active {
            transform: scale(3);
            color: white;
        }
    }
    .next-btn.gold, .last-btn.gold {
        &:hover {
            transform: scale(2.5);
            color: $goldDark;
        }
    }
    .next-btn.crimson, .last-btn.crimson {
        &:hover {
            transform: scale(2.5);
            color: $crimsonBright;
            text-shadow: 0 0 10px $crimsonDark;
        }
    }
    @keyframes fadeIn {
        0% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
    #carousel.back-light {
        .slide {
            filter: drop-shadow(0 0 6px $crimsonDark) brightness(0.6);
        }
        .slide.active {
            filter: drop-shadow(0 0 8px $crimsonDark);

            &:hover {
                filter: drop-shadow(0 0 8px $crimsonBright);
            }
        }
    }
    .slide {
        cursor: pointer;
        min-width: 100%;
        aspect-ratio: 16/9;
        transition: 0.25s ease-in-out;
        filter: brightness(0.45);
        transform: scale(0.8) translateZ(0);
        display: flex;
        align-items: center;
        justify-content: flex-end;
        user-select: none;
        flex-direction: column;
        pointer-events: none;

        h2 {
            width: 80%;
            font-family: 'Noto Serif Georgian', sans-serif;
            font-size: 1.8rem;
            margin-bottom: 1rem;
            color: white;
            font-weight: bold;
            text-align: center;
        }
        h3 {
            max-width: 50%;
            font-family: 'Quicksand', sans-serif;
            font-size: 1rem;
            padding: 0.2rem 0.4rem;
            font-weight: 500;
            text-align: center;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;

            span {
                &:last-child {
                    &::after {
                        content: "";
                    }
                }
                &::after {
                    content: ", ";
                }
            }
        }
        h3.gold {
            color: black;
            background-color: $goldDark;
        }
        h3.crimson {
            color: white;
            background-color: $crimsonDark;
        }
    }
    .slide.active {
        transform: scale(1.4) perspective(1px) translateZ(0);
        filter: brightness(1);
        margin: 5vh 5vw;
        opacity: 1;
        pointer-events: auto;
    }
</style>