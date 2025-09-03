<script>
    import { onMount } from 'svelte';
    import { fade, scale, fly } from 'svelte/transition';

    let allPokemonData = [];
    let filteredPokemon = [];
    let searchTerm = '';
    let selectedPokemon = null;
    let loading = true;
    let showScrollIndicator = true;
    
    let displayedText = '';
    let fullText = 'Explore o mundo dos Pokémons da primeira geração';
    let typingSpeed = 50;

    async function typeWriter() {
        for (let i = 0; i <= fullText.length; i++) {
            displayedText = fullText.substring(0, i);
            await new Promise(resolve => setTimeout(resolve, typingSpeed));
        }
    }

    const typeColorMapping = {
        normal: '#A8A878', grass: '#78C850', fire: '#F08030', water: '#6890F0',
        electric: '#F8D030', ice: '#98D8D8', fighting: '#C03028', poison: '#A040A0',
        ground: '#E0C068', flying: '#A890F0', psychic: '#F85888', bug: '#A8B820',
        rock: '#B8A038', ghost: '#705898', dragon: '#7038F8', dark: '#705848',
        steel: '#B8B8D0', fairy: '#EE99AC'
    };

    onMount(async () => {
        try {
            const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=151');
            const data = await response.json();
            const pokemonDetails = await Promise.all(
                data.results.map((pokemon) => fetch(pokemon.url).then((res) => res.json()))
            );
            allPokemonData = pokemonDetails;
            typeWriter();
        } catch (error) {
            console.error('Erro ao buscar os Pokémon:', error);
        } finally {
            loading = false;
            setTimeout(() => showScrollIndicator = false, 3000);
        }
    });

    $: filteredPokemon = allPokemonData.filter(
        (p) =>
            p.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
            p.id.toString().includes(searchTerm)
    );

    function openModal(pokemon) {
        selectedPokemon = pokemon;
    }

    function closeModal() {
        selectedPokemon = null;
    }
</script>

<svelte:head>
    <title>Pokédex Animada</title>
    <link rel="icon" href="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/items/poke-ball.png" />
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">
</svelte:head>

<div class="container mx-auto p-4 md:p-8 text-gray-800 min-h-screen">
    <header class="text-center mb-8" in:fly={{ y: -50, duration: 800 }}>
        <div class="flex items-center justify-center gap-4">
            <img
                src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/25.png"
                alt="Pikachu Logo"
                class="h-16 md:h-20 animate-bounce"
            />
            <h1 class="text-4xl md:text-5xl font-bold tracking-tight text-gray-900 animate-pulse">Pokédex</h1>
        </div>
        <p class="text-gray-600 mt-2 min-h-[2rem]">{displayedText}<span class="typing-cursor">|</span></p>
    </header>

    <div class="mb-8 max-w-lg mx-auto" in:fly={{ y: 50, duration: 800, delay: 200 }}>
        <input
            bind:value={searchTerm}
            type="text"
            placeholder="Pesquisar por nome ou número..."
            class="w-full px-4 py-3 bg-white border-2 border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-yellow-500 focus:border-transparent transition-all duration-300 shadow-md hover:shadow-lg"
        />
    </div>

    {#if loading}
        <div class="text-center py-10" in:fade={{ duration: 500 }}>
            <div class="pokeball-loading">
                <div class="pokeball"></div>
            </div>
            <p class="text-lg font-semibold text-gray-700 mt-4">Carregando Pokémon...</p>
        </div>
    {:else if filteredPokemon.length === 0}
        <div class="text-center text-gray-500 text-xl py-10" in:fade>
            <p>Nenhum Pokémon encontrado. Tente outra busca!</p>
        </div>
    {:else}
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
            {#each filteredPokemon as pokemon, index (pokemon.id)}
                <button
                    class="pokemon-card text-white rounded-lg p-4 cursor-pointer relative overflow-hidden group w-full text-left"
                    style="background-color: {typeColorMapping[pokemon.types[0].type.name]};"
                    on:click={() => openModal(pokemon)}
                    in:fly={{ y: 100, duration: 600, delay: index * 50 }}
                    out:scale={{ duration: 300 }}
                >
                    <div class="pokeball-bg" />
                    <div class="relative z-10 flex justify-between items-start">
                        <div class="flex flex-col">
                            <span class="font-bold text-2xl capitalize transition-all duration-300 group-hover:translate-x-1">{pokemon.name}</span>
                            <div class="flex flex-col items-start gap-1 mt-2">
                                {#each pokemon.types as typeInfo}
                                    <span class="px-3 py-1 text-sm font-semibold text-gray-800 bg-white bg-opacity-30 rounded-full transition-all duration-300 group-hover:bg-opacity-40">
                                        {typeInfo.type.name}
                                    </span>
                                {/each}
                            </div>
                        </div>
                        <span class="font-bold text-lg opacity-50 transition-all duration-300 group-hover:opacity-70">#{pokemon.id.toString().padStart(3, '0')}</span>
                    </div>
                    <div class="relative z-10 text-right -mb-4 -mr-4 transition-all duration-500 group-hover:-translate-y-2">
                        <img
                            src={pokemon.sprites.other['official-artwork'].front_default}
                            alt={pokemon.name}
                            class="w-32 h-32 ml-auto transition-all duration-300 group-hover:scale-110"
                            loading="lazy"
                        />
                    </div>
                </button>
            {/each}
        </div>
        
        {#if showScrollIndicator && filteredPokemon.length > 8}
            <div class="flex justify-center mt-8 animate-bounce" in:fade={{ delay: 2000 }}>
                <div class="bg-gray-800 text-white px-4 py-2 rounded-full text-sm">
                    Scroll para ver mais
                </div>
            </div>
        {/if}
    {/if}
</div>

{#if selectedPokemon}
    <div 
        class="fixed inset-0 bg-black bg-opacity-80 flex justify-center items-center p-4 z-50" 
        on:click={closeModal}
        transition:fade={{ duration: 300 }}
    >
        <div 
            class="w-full max-w-lg rounded-2xl shadow-xl text-white relative overflow-hidden" 
            on:click|stopPropagation
            style="background: linear-gradient(135deg, {typeColorMapping[selectedPokemon.types[0].type.name]} 0%, {typeColorMapping[selectedPokemon.types[selectedPokemon.types.length-1]?.type.name] || typeColorMapping[selectedPokemon.types[0].type.name]} 100%)"
            in:scale={{ duration: 400, delay: 100 }}
            out:fade={{ duration: 200 }}
        >
            <button on:click={closeModal} class="absolute top-4 right-4 text-white text-3xl hover:text-gray-300 transition z-10 bg-black bg-opacity-30 rounded-full w-10 h-10 flex items-center justify-center transform hover:rotate-90 duration-300">&times;</button>
            
            <div class="modal-header-bg p-8 rounded-t-2xl relative overflow-hidden">
                <div class="pokeball-bg opacity-15" style="background-size: 220px; background-position: right -30px center;"></div>
                <div class="relative z-10">
                    <div class="flex justify-between items-center">
                        <h2 class="text-4xl font-bold capitalize drop-shadow-md animate-pulse">{selectedPokemon.name}</h2>
                        <span class="text-2xl font-bold drop-shadow-md">#{selectedPokemon.id.toString().padStart(3, '0')}</span>
                    </div>
                    <div class="flex gap-2 mt-3">
                        {#each selectedPokemon.types as typeInfo}
                            <span class="px-4 py-1.5 text-sm font-semibold text-gray-800 bg-white bg-opacity-30 rounded-full backdrop-blur-sm border border-white border-opacity-20 drop-shadow transition-all duration-300 hover:scale-105">
                                {typeInfo.type.name}
                            </span>
                        {/each}
                    </div>
                </div>
            </div>

            <div class="bg-white p-6 pb-8 rounded-b-2xl text-gray-800 relative">
                <div class="absolute -top-28 left-1/2 -translate-x-1/2 h-56 pokemon-image-container">
                    <img
                        src={selectedPokemon.sprites.other['official-artwork'].front_default}
                        alt={selectedPokemon.name}
                        class="h-full object-contain drop-shadow-2xl pokemon-image"
                    />
                </div>
                
                <div class="mt-24">
                    <div class="grid grid-cols-2 gap-4 text-center mb-6">
                        <div class="bg-gray-50 p-3 rounded-xl shadow-sm transition-all duration-300 hover:shadow-md">
                            <h4 class="text-sm font-bold text-gray-500 mb-1">Height</h4>
                            <p class="text-lg font-semibold">{(selectedPokemon.height / 10).toFixed(2)} m</p>
                        </div>
                        <div class="bg-gray-50 p-3 rounded-xl shadow-sm transition-all duration-300 hover:shadow-md">
                            <h4 class="text-sm font-bold text-gray-500 mb-1">Weight</h4>
                            <p class="text-lg font-semibold">{(selectedPokemon.weight / 10).toFixed(2)} kg</p>
                        </div>
                        <div class="col-span-2 bg-gray-50 p-3 rounded-xl shadow-sm transition-all duration-300 hover:shadow-md">
                            <h4 class="text-sm font-bold text-gray-500 mb-1">Abilities</h4>
                            <p class="text-lg font-semibold capitalize">
                                {selectedPokemon.abilities.map(a => a.ability.name).join(', ')}
                            </p>
                        </div>
                    </div>

                    <h3 class="text-center text-xl font-bold mb-4 border-t pt-4 border-gray-200">Base Stats</h3>
                    <div class="space-y-3">
                        {#each selectedPokemon.stats as s, i}
                            <div class="flex items-center gap-3" in:fly={{ y: 20, duration: 500, delay: i * 100 }}>
                                <span class="w-1/4 text-right font-semibold capitalize text-sm text-gray-600">{s.stat.name.replace('-', ' ')}</span>
                                <span class="w-10 text-left font-bold text-gray-800">{s.base_stat}</span>
                                <div class="w-full bg-gray-200 rounded-full h-2.5 overflow-hidden">
                                    <div
                                        class="stat-bar h-2.5 rounded-full"
                                        style="width: {(s.base_stat / 255) * 100}%; background-color: {typeColorMapping[selectedPokemon.types[0].type.name]};"
                                    />
                                </div>
                            </div>
                        {/each}
                    </div>
                </div>
            </div>
        </div>
    </div>
{/if}

<style>
    :global(body) {
        font-family: 'Poppins', sans-serif;
        background-color: #f0f2f5;
        overflow-x: hidden;
    }

    .typing-cursor {
        animation: blink 1s infinite;
    }
    @keyframes blink {
        0%, 50% { opacity: 1; }
        51%, 100% { opacity: 0; }
    }

    .pokeball-loading {
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .pokeball {
        width: 60px;
        height: 60px;
        background-color: #fff;
        border-radius: 50%;
        position: relative;
        overflow: hidden;
        border: 3px solid;
        animation: rotate 1.5s linear infinite;
    }
    .pokeball:before {
        content: "";
        position: absolute;
        background-color: red;
        width: 100%;
        height: 50%;
    }
    .pokeball:after {
        content: "";
        position: absolute;
        top: calc(50% - 3px);
        width: 100%;
        height: 6px;
        background-color: black;
    }
    @keyframes rotate {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
    }

    .pokemon-card {
        transition: transform 0.3s ease, box-shadow 0.3s ease;
        position: relative;
        overflow: hidden;
    }
    .pokemon-card:hover {
        transform: translateY(-8px) scale(1.02);
        box-shadow: 0 15px 30px rgba(0,0,0,0.2);
    }
    .pokeball-bg {
        background-image: url('https://upload.wikimedia.org/wikipedia/commons/5/53/Pok%C3%A9_Ball_icon.svg');
        background-repeat: no-repeat;
        background-position: right -20px top -20px;
        background-size: 120px;
        opacity: 0.2;
        position: absolute;
        inset: 0;
        transition: all 0.5s ease;
    }
    .pokemon-card:hover .pokeball-bg {
        transform: rotate(15deg) scale(1.1);
        opacity: 0.25;
    }

    .stat-bar {
        transition: width 1s cubic-bezier(0.22, 0.61, 0.36, 1);
    }
    .modal-header-bg::before {
        content: '';
        position: absolute;
        inset: 0;
        background: repeating-linear-gradient(-45deg, rgba(255,255,255,0.1), rgba(255,255,255,0.1) 10px, transparent 10px, transparent 20px);
        opacity: 0.5;
    }
    .pokemon-image {
        filter: drop-shadow(0 10px 15px rgba(0, 0, 0, 0.2));
        transition: transform 0.5s ease;
    }
    .pokemon-image-container:hover .pokemon-image {
        transform: scale(1.08) translateY(-5px);
    }
</style>

