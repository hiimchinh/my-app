<template>
  <div id="app">
    <label>
      Search Crocodillians:
      <input v-model="query"/>
    </label>
    <table style="width: 100%">
      <thead>
        <th>Name</th>
        <th>Age</th>
        <th>Species</th>
        <th>Country</th>
        <th>Likes to Eat</th>
      </thead>
      <tbody>
        <tr v-for="result of queryResults">
          <td>{{result.name}}</td>
          <td>{{result.age}}</td>
          <td>{{result.species}}</td>
          <td>{{result.country}}</td>
          <td>{{result.eats}}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import fz from 'fuzzaldrin-plus';
import Crocodilians from './crocodilians.json';

export default {
  name: 'app',
  data() {
    return {
      query: '',
      options: Crocodilians
    }
  },

  computed: {
    queryResults() {
      if(!this.query) return this.options;

      const preparedQuery = fz.prepareQuery(this.query);
      const scores = {};

      return this.options
        .map((option, index) => {
          const scorableFields = [
            option.uuid,
            option.name,
            option.species,
            option.country,
            option.eats,
            `${option.age}`,
          ].map(toScore => fz.score(toScore, this.query, { preparedQuery }));

          scores[option.uuid] = Math.max(...scorableFields);

          return option;
        })
        .filter(option => scores[option.uuid] > 1)
        .sort((a, b) => scores[b.uuid] - scores[a.uuid])
      ;
    }
  }
}
</script>