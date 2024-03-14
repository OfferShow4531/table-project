<template>
  <div>
    <header>
      <h1>Проведение ТО и мелкий ремонт</h1>
    </header>

    <main>
      <div class="panel">
        <div class="side-menu"></div>
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
</style>