import './styles.css';

// ===================================
// TURBOFLIX - MAIN APPLICATION
// ===================================

// === CONFIGURATION ===
const CONFIG = {
  TV_JSON_URL: 'https://7-7-7-15.github.io/Tv.json',
  MOVIE_JSON_URL: 'https://7-7-7-15.github.io/Movies.json',
  POSTER_BASE_URL: 'https://image.tmdb.org/t/p/w500',
  VIDEO_BASE_URL: 'https://turbovid.eu/api/req',
  MAX_ITEMS: 20,
  SEARCH_DEBOUNCE: 300,
};

// === STATE ===
const state = {
  allContent: [] as Content[],
  tvShows: [] as Content[],
  movies: [] as Content[],
  myList: [] as Content[],
  continueWatching: [] as Content[],
  currentCategory: 'all',
  searchQuery: '',
  isLoading: false,
};

// === TYPES ===
interface Content {
  title: string;
  tmdb: string;
  type: 'tv' | 'movie';
  category?: string;
}

// === INIT ===
document.addEventListener('DOMContentLoaded', () => {
  setTimeout(() => {
    hideSplashScreen();
    initializeApp();
  }, 3500);
});

function hideSplashScreen() {
  const splash = document.getElementById('netflix-intro');
  if (splash) {
    splash.style.display = 'none';
  }
}

async function initializeApp() {
  setupEventListeners();
  await loadContent();
  renderHeroBanner();
  renderAllContent();
  loadLocalData();
}

// === DATA LOADING ===
async function loadContent() {
  try {
    state.isLoading = true;

    const [tvData, movieData] = await Promise.all([
      fetch(CONFIG.TV_JSON_URL).then((r) => r.json()),
      fetch(CONFIG.MOVIE_JSON_URL).then((r) => r.json()),
    ]);

    state.tvShows = tvData.map((item: any) => ({
      ...item,
      type: 'tv' as const,
      category: 'TV Shows',
    }));

    state.movies = movieData.map((item: any) => ({
      ...item,
      type: 'movie' as const,
      category: 'Movies',
    }));

    state.allContent = [...state.tvShows, ...state.movies];
    state.isLoading = false;
  } catch (error) {
    console.error('Error loading content:', error);
    state.isLoading = false;
  }
}

function loadLocalData() {
  const savedList = localStorage.getItem('myList');
  const savedContinue = localStorage.getItem('continueWatching');

  if (savedList) {
    state.myList = JSON.parse(savedList);
    if (state.myList.length > 0) {
      renderMyList();
    }
  }

  if (savedContinue) {
    state.continueWatching = JSON.parse(savedContinue);
    if (state.continueWatching.length > 0) {
      renderContinueWatching();
    }
  }
}

function saveToMyList(item: Content) {
  const exists = state.myList.some((i) => i.tmdb === item.tmdb);
  if (!exists) {
    state.myList.push(item);
    localStorage.setItem('myList', JSON.stringify(state.myList));
    renderMyList();
  }
}

function removeFromMyList(item: Content) {
  state.myList = state.myList.filter((i) => i.tmdb !== item.tmdb);
  localStorage.setItem('myList', JSON.stringify(state.myList));
  renderMyList();
}

function addToContinueWatching(item: Content) {
  const exists = state.continueWatching.some((i) => i.tmdb === item.tmdb);
  if (!exists) {
    state.continueWatching = [item, ...state.continueWatching.slice(0, 9)];
    localStorage.setItem('continueWatching', JSON.stringify(state.continueWatching));
    renderContinueWatching();
  }
}

// === EVENT LISTENERS ===
function setupEventListeners() {
  // Header scroll effect
  window.addEventListener('scroll', handleScroll);

  // Search
  const searchToggle = document.getElementById('search-toggle');
  const searchBox = document.getElementById('search-box');
  const searchInput = document.getElementById('search-input') as HTMLInputElement;

  searchToggle?.addEventListener('click', () => {
    searchBox?.classList.toggle('active');
    if (searchBox?.classList.contains('active')) {
      searchInput?.focus();
    }
  });

  let searchTimeout: number;
  searchInput?.addEventListener('input', (e) => {
    clearTimeout(searchTimeout);
    searchTimeout = window.setTimeout(() => {
      state.searchQuery = (e.target as HTMLInputElement).value.toLowerCase();
      handleSearch();
    }, CONFIG.SEARCH_DEBOUNCE);
  });

  // Navigation
  const navLinks = document.querySelectorAll('.nav-link');
  navLinks.forEach((link) => {
    link.addEventListener('click', (e) => {
      e.preventDefault();
      navLinks.forEach((l) => l.classList.remove('active'));
      link.classList.add('active');
      const category = link.getAttribute('data-category') || 'all';
      state.currentCategory = category;
      filterByCategory(category);
    });
  });

  // Hero actions
  document.getElementById('hero-play')?.addEventListener('click', () => {
    if (state.allContent.length > 0) {
      playContent(state.allContent[0]);
    }
  });

  document.getElementById('hero-more-info')?.addEventListener('click', () => {
    if (state.allContent.length > 0) {
      showInfoModal(state.allContent[0]);
    }
  });

  // Mute toggle
  const muteToggle = document.getElementById('mute-toggle');
  muteToggle?.addEventListener('click', () => {
    const volumeOn = document.getElementById('volume-on');
    const volumeOff = document.getElementById('volume-off');
    if (volumeOn && volumeOff) {
      if (volumeOn.style.display === 'none') {
        volumeOn.style.display = 'block';
        volumeOff.style.display = 'none';
      } else {
        volumeOn.style.display = 'none';
        volumeOff.style.display = 'block';
      }
    }
  });

  // Modal close buttons
  document.getElementById('player-close')?.addEventListener('click', closePlayer);
  document.getElementById('info-close')?.addEventListener('click', closeInfoModal);

  // Close modals on escape
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
      closePlayer();
      closeInfoModal();
    }
  });

  // Close modals on background click
  document.getElementById('player-modal')?.addEventListener('click', (e) => {
    if ((e.target as HTMLElement).id === 'player-modal') {
      closePlayer();
    }
  });

  document.getElementById('info-modal')?.addEventListener('click', (e) => {
    if ((e.target as HTMLElement).id === 'info-modal') {
      closeInfoModal();
    }
  });
}

function handleScroll() {
  const header = document.getElementById('header');
  if (window.scrollY > 50) {
    header?.classList.add('scrolled');
  } else {
    header?.classList.remove('scrolled');
  }
}

function handleSearch() {
  if (state.searchQuery === '') {
    renderAllContent();
  } else {
    const filtered = state.allContent.filter((item) =>
      item.title.toLowerCase().includes(state.searchQuery)
    );
    renderSearchResults(filtered);
  }
}

function filterByCategory(category: string) {
  const categoryRows = document.getElementById('category-rows');
  const top10Section = document.getElementById('top10-section');
  const trendingSection = document.getElementById('trending-section');
  const myListSection = document.getElementById('my-list-section');

  if (category === 'all') {
    renderAllContent();
    if (top10Section) top10Section.style.display = 'block';
    if (trendingSection) trendingSection.style.display = 'block';
    if (myListSection) myListSection.style.display = state.myList.length > 0 ? 'block' : 'none';
  } else if (category === 'tv') {
    if (categoryRows) categoryRows.innerHTML = '';
    renderRow('TV Shows', state.tvShows.slice(0, CONFIG.MAX_ITEMS), 'category-rows');
    if (top10Section) top10Section.style.display = 'none';
    if (trendingSection) trendingSection.style.display = 'none';
    if (myListSection) myListSection.style.display = 'none';
  } else if (category === 'movies') {
    if (categoryRows) categoryRows.innerHTML = '';
    renderRow('Movies', state.movies.slice(0, CONFIG.MAX_ITEMS), 'category-rows');
    if (top10Section) top10Section.style.display = 'none';
    if (trendingSection) trendingSection.style.display = 'none';
    if (myListSection) myListSection.style.display = 'none';
  } else if (category === 'new') {
    if (categoryRows) categoryRows.innerHTML = '';
    const newContent = [...state.allContent].slice(0, CONFIG.MAX_ITEMS);
    renderRow('New & Popular', newContent, 'category-rows');
    if (top10Section) top10Section.style.display = 'none';
    if (trendingSection) trendingSection.style.display = 'none';
    if (myListSection) myListSection.style.display = 'none';
  } else if (category === 'mylist') {
    if (categoryRows) categoryRows.innerHTML = '';
    if (top10Section) top10Section.style.display = 'none';
    if (trendingSection) trendingSection.style.display = 'none';
    if (myListSection) myListSection.style.display = state.myList.length > 0 ? 'block' : 'none';
  }
}

// === RENDERING ===
function renderHeroBanner() {
  if (state.allContent.length === 0) return;

  const hero = state.allContent[Math.floor(Math.random() * Math.min(20, state.allContent.length))];
  const banner = document.getElementById('hero-banner');
  const background = banner?.querySelector('.hero-background') as HTMLElement;

  if (background) {
    background.style.backgroundImage = `url(${getPosterUrl(hero.tmdb)})`;
  }

  const title = banner?.querySelector('.hero-title');
  if (title) title.textContent = hero.title;

  const match = banner?.querySelector('.hero-match');
  if (match) match.textContent = '98% Match';

  const year = banner?.querySelector('.hero-year');
  if (year) year.textContent = '2024';

  const rating = banner?.querySelector('.hero-rating');
  if (rating) rating.textContent = 'PG-13';

  const seasons = banner?.querySelector('.hero-seasons');
  if (seasons && hero.type === 'tv') {
    seasons.textContent = '3 Seasons';
  }

  const description = banner?.querySelector('.hero-description');
  if (description) {
    description.textContent = getRandomDescription();
  }
}

function renderAllContent() {
  renderTop10();
  renderTrending();

  const categoryRows = document.getElementById('category-rows');
  if (categoryRows) {
    categoryRows.innerHTML = '';
    renderRow('Popular on TurboFlix', state.allContent.slice(0, CONFIG.MAX_ITEMS), 'category-rows');
    renderRow('Action & Adventure', state.movies.slice(5, 5 + CONFIG.MAX_ITEMS), 'category-rows');
    renderRow('Drama Series', state.tvShows.slice(10, 10 + CONFIG.MAX_ITEMS), 'category-rows');
  }
}

function renderTop10() {
  const container = document.getElementById('top10-row');
  if (!container) return;

  container.innerHTML = '';
  const top10Items = state.allContent.slice(0, 10);

  top10Items.forEach((item, index) => {
    const card = createTop10Card(item, index + 1);
    container.appendChild(card);
  });
}

function renderTrending() {
  const container = document.getElementById('trending-row');
  if (!container) return;

  const trending = [...state.allContent].slice(10, 10 + CONFIG.MAX_ITEMS);
  container.innerHTML = '';
  trending.forEach((item) => {
    const card = createContentCard(item);
    container.appendChild(card);
  });
}

function renderMyList() {
  const section = document.getElementById('my-list-section');
  const container = document.getElementById('my-list-row');
  if (!container || !section) return;

  if (state.myList.length === 0) {
    section.style.display = 'none';
    return;
  }

  section.style.display = 'block';
  container.innerHTML = '';
  state.myList.forEach((item) => {
    const card = createContentCard(item);
    container.appendChild(card);
  });
}

function renderContinueWatching() {
  const section = document.getElementById('continue-watching-section');
  const container = document.getElementById('continue-watching-row');
  if (!container || !section) return;

  if (state.continueWatching.length === 0) {
    section.style.display = 'none';
    return;
  }

  section.style.display = 'block';
  container.innerHTML = '';
  state.continueWatching.forEach((item) => {
    const card = createContentCard(item);
    container.appendChild(card);
  });
}

function renderRow(title: string, items: Content[], containerId: string) {
  const container = document.getElementById(containerId);
  if (!container) return;

  const section = document.createElement('section');
  section.className = 'content-section';

  const titleEl = document.createElement('h2');
  titleEl.className = 'section-title';
  titleEl.textContent = title;

  const row = document.createElement('div');
  row.className = 'content-row';

  items.forEach((item) => {
    const card = createContentCard(item);
    row.appendChild(card);
  });

  section.appendChild(titleEl);
  section.appendChild(row);
  container.appendChild(section);
}

function renderSearchResults(results: Content[]) {
  const categoryRows = document.getElementById('category-rows');
  const top10Section = document.getElementById('top10-section');
  const trendingSection = document.getElementById('trending-section');

  if (top10Section) top10Section.style.display = 'none';
  if (trendingSection) trendingSection.style.display = 'none';

  if (categoryRows) {
    categoryRows.innerHTML = '';
    if (results.length > 0) {
      renderRow(`Search Results (${results.length})`, results.slice(0, CONFIG.MAX_ITEMS), 'category-rows');
    } else {
      categoryRows.innerHTML = '<div style="text-align:center; padding:4rem; color:#808080;">No results found</div>';
    }
  }
}

// === CARD CREATORS ===
function createContentCard(item: Content): HTMLElement {
  const card = document.createElement('div');
  card.className = 'content-card';
  card.style.backgroundImage = `url(${getPosterUrl(item.tmdb)})`;

  const info = document.createElement('div');
  info.className = 'card-info';

  const title = document.createElement('div');
  title.className = 'card-title';
  title.textContent = item.title;

  const meta = document.createElement('div');
  meta.className = 'card-meta';
  meta.innerHTML = `
    <span class="card-match">98% Match</span>
    <span>${item.type === 'tv' ? 'Series' : 'Movie'}</span>
  `;

  info.appendChild(title);
  info.appendChild(meta);

  const actions = document.createElement('div');
  actions.className = 'card-actions';

  const addBtn = document.createElement('button');
  addBtn.className = 'card-action-btn';
  addBtn.innerHTML = '+';
  addBtn.onclick = (e) => {
    e.stopPropagation();
    saveToMyList(item);
  };

  const likeBtn = document.createElement('button');
  likeBtn.className = 'card-action-btn';
  likeBtn.innerHTML = '👍';
  likeBtn.onclick = (e) => {
    e.stopPropagation();
  };

  actions.appendChild(addBtn);
  actions.appendChild(likeBtn);

  const playIcon = document.createElement('div');
  playIcon.className = 'card-play-icon';
  playIcon.innerHTML = '<svg viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>';

  card.appendChild(info);
  card.appendChild(actions);
  card.appendChild(playIcon);

  card.onclick = () => playContent(item);

  return card;
}

function createTop10Card(item: Content, rank: number): HTMLElement {
  const card = document.createElement('div');
  card.className = 'top10-card';

  const number = document.createElement('div');
  number.className = 'top10-number';
  number.textContent = rank.toString();

  const poster = document.createElement('div');
  poster.className = 'top10-poster';
  poster.style.backgroundImage = `url(${getPosterUrl(item.tmdb)})`;

  card.appendChild(number);
  card.appendChild(poster);

  card.onclick = () => playContent(item);

  return card;
}

// === MODALS ===
function playContent(item: Content) {
  const modal = document.getElementById('player-modal');
  const iframe = document.getElementById('player-iframe') as HTMLIFrameElement;
  const title = document.getElementById('player-title');
  const match = document.getElementById('player-match');
  const year = document.getElementById('player-year');
  const rating = document.getElementById('player-rating');
  const description = document.getElementById('player-description');
  const cast = document.getElementById('player-cast');
  const genres = document.getElementById('player-genres');

  if (!modal || !iframe) return;

  const videoUrl = getVideoUrl(item);
  iframe.src = videoUrl;

  if (title) title.textContent = item.title;
  if (match) match.textContent = '98% Match';
  if (year) year.textContent = '2024';
  if (rating) rating.textContent = 'PG-13';
  if (description) description.textContent = getRandomDescription();
  if (cast) cast.textContent = 'John Doe, Jane Smith, Bob Johnson';
  if (genres) genres.textContent = 'Action, Drama, Thriller';

  modal.classList.add('active');
  document.body.style.overflow = 'hidden';

  addToContinueWatching(item);
}

function closePlayer() {
  const modal = document.getElementById('player-modal');
  const iframe = document.getElementById('player-iframe') as HTMLIFrameElement;

  if (!modal) return;

  modal.classList.remove('active');
  iframe.src = '';
  document.body.style.overflow = '';
}

function showInfoModal(item: Content) {
  const modal = document.getElementById('info-modal');
  const heroBg = document.getElementById('info-hero-bg') as HTMLElement;
  const title = document.getElementById('info-title');
  const match = document.getElementById('info-match');
  const year = document.getElementById('info-year');
  const rating = document.getElementById('info-rating');
  const description = document.getElementById('info-description');
  const cast = document.getElementById('info-cast');
  const genres = document.getElementById('info-genres');
  const tags = document.getElementById('info-tags');

  if (!modal) return;

  if (heroBg) heroBg.style.backgroundImage = `url(${getPosterUrl(item.tmdb)})`;
  if (title) title.textContent = item.title;
  if (match) match.textContent = '98% Match';
  if (year) year.textContent = '2024';
  if (rating) rating.textContent = 'PG-13';
  if (description) description.textContent = getRandomDescription();
  if (cast) cast.textContent = 'John Doe, Jane Smith, Bob Johnson';
  if (genres) genres.textContent = 'Action, Drama, Thriller';
  if (tags) tags.textContent = 'Exciting, Suspenseful, Award-winning';

  // Setup play button
  const playBtn = document.getElementById('info-play');
  if (playBtn) {
    playBtn.onclick = () => {
      closeInfoModal();
      playContent(item);
    };
  }

  // Setup add to list button
  const addBtn = document.getElementById('info-add-list');
  if (addBtn) {
    const isInList = state.myList.some((i) => i.tmdb === item.tmdb);
    if (isInList) {
      addBtn.classList.add('active');
      addBtn.innerHTML = '<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/></svg>';
      addBtn.onclick = () => {
        removeFromMyList(item);
        addBtn.classList.remove('active');
        addBtn.innerHTML = '<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>';
      };
    } else {
      addBtn.classList.remove('active');
      addBtn.innerHTML = '<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>';
      addBtn.onclick = () => {
        saveToMyList(item);
        addBtn.classList.add('active');
        addBtn.innerHTML = '<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/></svg>';
      };
    }
  }

  modal.classList.add('active');
  document.body.style.overflow = 'hidden';
}

function closeInfoModal() {
  const modal = document.getElementById('info-modal');
  if (!modal) return;

  modal.classList.remove('active');
  document.body.style.overflow = '';
}

// === UTILITIES ===
function getPosterUrl(tmdb: string): string {
  return `${CONFIG.POSTER_BASE_URL}/${tmdb}.jpg`;
}

function getVideoUrl(item: Content): string {
  const suffix = item.type === 'movie' ? '' : '/1/1';
  return `${CONFIG.VIDEO_BASE_URL}/${item.type}/${item.tmdb}${suffix}`;
}

function getRandomDescription(): string {
  const descriptions = [
    'An epic journey filled with suspense, drama, and unforgettable moments that will keep you on the edge of your seat.',
    'A gripping tale of courage, love, and redemption that explores the depths of human emotion and resilience.',
    'Experience the thrill of adventure in this action-packed story that combines stunning visuals with compelling characters.',
    'A masterfully crafted narrative that weaves together multiple storylines into an unforgettable cinematic experience.',
    'This critically acclaimed series delivers powerful performances and thought-provoking themes that resonate long after viewing.',
  ];
  return descriptions[Math.floor(Math.random() * descriptions.length)];
}
