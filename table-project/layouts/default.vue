<template>
  <div>
    <header>
      <h1>Проведение ТО и мелкий ремонт</h1>
    </header>

    <main>
      <div class="panel">
        <div class="side-menu"></div>
        <p class="mobile-text">Проведение ТО и мелкий ремонт</p>
        <slot />
      </div>
    </main>

    <footer>
      <h2>Тренировочный макет 2024</h2>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'DefaultLayout'
}
</script>

<style>
/* Здесь могут быть ваши стили для макета по умолчанию */
header {
  background-image: radial-gradient(circle at 29% 0, #000, #1c2734 103%);
  color: white;
}

h1{
  margin-block-start: 0;
  margin-block-end: 0;
}

h2{
  margin-block-start: 0;
  margin-block-end: 0;
}

.mobile-text{
  display: none;
}

.panel{
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: row;
}

.side-menu{
    width: 229px;
    height: 860px;
    margin: 0 25px 0 0;
    background-image: radial-gradient(circle at 29% 0, #000, #1c2734 103%);
}

footer {
  background-image: radial-gradient(circle at 29% 0, #000, #1c2734 103%);
  color: white;
}

@media (max-width: 768px) {
  header{
    display: none;
  }
  footer{
    display: none;
  }
  .side-menu{
    display: none;
  }
  .mobile-text{
    display: block;
    margin-block-start: 1.5625rem;
    margin-block-end: 1.5625rem;
    padding: 0.625rem;
  }
  .panel{
    display: block;
  }
}
</style>