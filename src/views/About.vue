<template>
  <div class="about">
    <h1>About the Twitter Poetry Generator</h1>
    <div class="container">
      <p>The Twitter Poetry Generator is a project by <a href="https://twitter.com/mperrien" target="_blank" rel="noopener">Marc Perrien</a>.</p>
      <p>It is my entry to <a href="https://poetryslash.com/en/index.html" target="_blank" rel="noopener">Poetry Slash 3</a>.</p>
      <p>Poetry Slash is a recital of computer-generated poetry that was supposed to take place in April 2020 in Madrid, during <a href="https://librosmutantes.com/" target="_blank" rel="noopener">Libros Mutantes</a>, Madrid's Art Book Fair. Obviously, Poetry Slash has been postponed, due to The Situation. <em>After</em> I bought my plane tickets, obviously.</p>
      <p><a href="https://github.com/mperrien/poetry-slash" target="_blank" rel="noopener">The code for the Twitter Poetry Generator is on GitHub</a>, although I wouldn't encourage anyone to read it, let alone use it as an example of how to write code. It's (poorly) written in <a href="https://vuejs.org/" taeget="_blank" rel="noopener">Vue.js</a>, using <a href="https://www.typescriptlang.org/" target="_blank" rel="noopener">TypeScript</a> and <a href="https://sass-lang.com/" target="_blank" rel="noopener">Sass</a>.</p>
      <p>The Twitter Poetry Generator is hosted on <a href="https://www.netlify.com/" target="_blank" rel="noopener">Netlify</a>.</p>
      <p>Stay home, wash your hands, and listen to Elliott Smith.</p>
      <p><router-link to="/">Go back to the home page</router-link></p>
    </div>
  </div>
</template>

<style lang="scss" scoped>
h1 {
  color: $violet-900;
  text-align: center;
}
</style>