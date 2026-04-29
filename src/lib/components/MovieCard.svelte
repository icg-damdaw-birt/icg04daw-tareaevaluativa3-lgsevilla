<script lang="ts">
  import type { Movie } from '$lib/types';
  import { moviesStore } from '$lib';

  // Props con Svelte 5: sistema de tipos explícito y callbacks en lugar de eventos
  let { 
    movie,
    showActions = true,
    ondelete,
    onedit
  }: {
    movie: Movie;
    showActions?: boolean;
    ondelete?: (id: string) => void;
    onedit?: (movie: Movie) => void;
  } = $props();

  // Estado local para toggle de favorito
  let togglingFavorite = $state(false);

  // Estado local para cambio de rating
  let togglingRating = $state(false);

  // Handlers: ejecutan callbacks del padre directamente
  function handleDelete() {
    ondelete?.(movie.id);
  }

  function handleEdit() {
    onedit?.(movie);
  }

  // Handler para marcar/desmarcar como favorito
  async function handleToggleFavorite() {
    togglingFavorite = true;
    const success = await moviesStore.toggleFavorite(movie.id);
    togglingFavorite = false;
  }

  // Handler para cambiar el rating
  async function handleRateMovie(rating: number) {
    togglingRating = true;
    const success = await moviesStore.rateMovie(movie.id, rating);
    togglingRating = false;
  }
</script>

<!-- Componente reutilizable: tarjeta para mostrar información de una película -->
<article class="flex flex-col overflow-hidden rounded-lg border border-slate-200 bg-white shadow-sm">
  {#if movie.posterUrl}
    <div class="flex h-48 items-center justify-center bg-slate-100">
      <img
        alt={`Póster de ${movie.title}`}
        class="max-h-full max-w-full object-contain"
        src={movie.posterUrl}
        loading="lazy"
      />
    </div>
  {/if}

  <div class="flex flex-1 flex-col gap-3 p-4">
    <header>
      <h3 class="text-lg font-semibold text-slate-900">{movie.title}</h3>
      <p class="text-sm text-slate-600">Dirigida por {movie.director}</p>
    </header>

    <div class="mt-auto text-sm text-slate-500">
      {#if movie.year}
        <span>Año: {movie.year}</span>
      {/if}
      {#if movie.isFavorite}
        <span class="ml-2 inline-block text-red-500">♥ Favorito</span>
      {/if}
    </div>

    <!-- Rating Stars (0-5) -->
    <div class="flex items-center gap-2">
      <div class="flex gap-1">
        {#each [1, 2, 3, 4, 5] as star}
          <button
            type="button"
            class="text-xl transition"
            class:text-yellow-400={star <= (movie.rating || 0)}
            class:text-gray-300={star > (movie.rating || 0)}
            class:hover:text-yellow-300={!togglingRating && star > (movie.rating || 0)}
            class:hover:text-yellow-500={!togglingRating && star <= (movie.rating || 0)}
            class:opacity-50={togglingRating}
            class:cursor-not-allowed={togglingRating}
            disabled={togglingRating}
            onclick={() => handleRateMovie(star)}
          >
            {star <= (movie.rating || 0) ? '⭐' : '☆'}
          </button>
        {/each}
      </div>
      {#if movie.rating}
        <span class="text-sm text-slate-600">{movie.rating}/5</span>
      {:else}
        <span class="text-sm text-slate-400">Sin calificar</span>
      {/if}
    </div>

    {#if showActions}
      <div class="mt-3 flex flex-col gap-2 sm:flex-row">
        <button
          type="button"
          class="rounded border px-3 py-2 transition"
          class:border-red-500={movie.isFavorite}
          class:text-red-600={movie.isFavorite}
          class:bg-red-50={movie.isFavorite && !togglingFavorite}
          class:hover:bg-red-100={movie.isFavorite}
          class:border-slate-300={!movie.isFavorite}
          class:text-slate-700={!movie.isFavorite}
          class:hover:bg-slate-50={!movie.isFavorite}
          class:opacity-50={togglingFavorite}
          class:cursor-not-allowed={togglingFavorite}
          disabled={togglingFavorite}
          onclick={handleToggleFavorite}
        >
          {#if togglingFavorite}
            Cambiando...
          {:else}
            {movie.isFavorite ? '♥ Favorito' : '♡ Agregar a favoritos'}
          {/if}
        </button>
        <button
          type="button"
          class="w-full rounded border border-slate-300 px-3 py-2 text-slate-700 transition hover:bg-slate-50"
          onclick={handleEdit}
        >
          Editar
        </button>
        <button
          type="button"
          class="w-full rounded border border-red-500 px-3 py-2 text-red-600 transition hover:bg-red-50"
          onclick={handleDelete}
        >
          Eliminar
        </button>
      </div>
    {/if}
  </div>
</article>
