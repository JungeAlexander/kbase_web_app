<template>
  <div class="home">
    <h1 class="display-2">Articles</h1>
    <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Quam quisquam doloribus cupiditate exercitationem repudiandae perspiciatis repellat veritatis consequatur, veniam eveniet asperiores, eos corporis maiores esse tempore illo voluptates optio amet.</p>
    <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Dicta, repudiandae! Aut minus numquam praesentium consectetur mollitia dignissimos totam blanditiis suscipit voluptatem! Quasi reprehenderit unde est minus mollitia voluptates nobis illum.</p>

    <v-btn dark color="pink">
      <v-icon left>mdi-text-subject</v-icon>
      <span>Show articles</span>
    </v-btn>

    <v-container class="my-5">
      <v-row justify-start class="mb-3">
        <v-col xs="12" md="3">
          <v-tooltip top>
            <template v-slot:activator="{ on }">
              <v-btn small text color="grey" v-on="on" @click="sortBy('title')">
                <v-icon small left>mdi-format-title</v-icon>
                <span class="caption text-lowercase">By title</span>
              </v-btn>
            </template>
            <span>Sort by project name</span>
          </v-tooltip>
        </v-col>
        <v-col xs="12" md="3">
          <v-tooltip top>
            <template v-slot:activator="{ on }">
              <v-btn small text color="grey" v-on="on" @click="sortBy('source')">
                <v-icon small left>mdi-source-branch</v-icon>
                <span class="caption text-lowercase">By Source</span>
              </v-btn>
            </template>
            <span>Sort by source</span>
          </v-tooltip>
        </v-col>
        <v-col xs="12" md="3">
          <v-tooltip top>
            <template v-slot:activator="{ on }">
              <v-btn small text color="grey" v-on="on" @click="sortBy('article_type')">
                <v-icon small left>mdi-format-list-bulleted-type</v-icon>
                <span class="caption text-lowercase">By Article type</span>
              </v-btn>
            </template>
            <span>Sort by article type</span>
          </v-tooltip>
        </v-col>
        <v-col xs="12" md="3">
          <v-tooltip top>
            <template v-slot:activator="{ on }">
              <v-btn small text color="grey" v-on="on" @click="sortBy('publication_date')">
                <v-icon small left>mdi-calendar-range</v-icon>
                <span class="caption text-lowercase">By Publication date</span>
              </v-btn>
            </template>
            <span>Sort by publication date</span>
          </v-tooltip>
        </v-col>
        <v-spacer></v-spacer>
      </v-row>

      <v-card flat v-for="article in articles" v-bind:key="article.id">
        <v-row wrap :class="`pa-3 article ${article.article_type}`">
          <v-col xs="12" md="6">
            <div class="caption grey--text">Article title</div>
            <div>{{ article.title }}</div>
          </v-col>
          <v-col xs="6" sm="4" md="1">
            <div class="caption grey--text">Source</div>
            <div>{{ article.source }}</div>
          </v-col>
          <v-col xs="6" sm="4" md="1">
            <div class="caption grey--text">Type</div>

            <div class="right">
              <v-chip
                small
                :color="article.article_type === 'preprint' ? '#3cd1c2': 'info'"
                class="white--text my-2 caption"
              >{{ article.article_type }}</v-chip>
            </div>
          </v-col>
          <v-col xs="6" sm="4" md="2">
            <div class="caption grey--text">Publication date</div>
            <div>{{ article.publication_date }}</div>
          </v-col>
          <v-col xs="3" sm="2" md="1">
            <div>
              <v-btn
                fab
                depressed
                small
                dark
                :outlined="user_liked_article(1, article.ratings)"
                color="light-blue darken-3"
                v-on:click="rate(article.id, 1, 1.0)"
              >
                <v-icon>mdi-thumb-up</v-icon>
              </v-btn>
            </div>
          </v-col>
          <v-col xs="3" sm="2" md="1">
            <div>
              <v-btn
                fab
                depressed
                small
                dark
                :outlined="user_disliked_article(1, article.ratings)"
                color="light-blue darken-3"
                v-on:click="rate(article.id, 1, -1.0)"
              >
                <v-icon>mdi-thumb-down</v-icon>
              </v-btn>
            </div>
          </v-col>
          <v-col xs="12" md="12">
            <div class="caption grey--text">Content</div>
            <div>{{ article.summary }}</div>
          </v-col>
          <v-col xs="12" md="12">
            <div class="caption grey--text">Authors</div>
            <div>{{ article.authors }}</div>
          </v-col>
        </v-row>
        <v-divider></v-divider>
      </v-card>
    </v-container>
  </div>
</template>

<script>
// @ is an alias to /src
import axios from "axios";

export default {
  name: "home",
  data: () => ({
    articles: []
  }),
  created() {
    fetch("http://kbase-nlb-c4b8ccc3f9b975ec.elb.eu-west-1.amazonaws.com/articles/?limit=10")
      .then(res => res.json())
      .then(response => {
        this.articles = response;
      });
    // .catch(e => {
    //   console.error(e.message);
    // });
  },
  methods: {
    sortBy(prop) {
      this.articles.sort((a, b) => (a[prop] < b[prop] ? -1 : 1));
    },
    rate: function(article, user, score) {
      //url = ;
      // json = {
      //          "value": score
      //        }
      axios
        .post("http://kbase-nlb-c4b8ccc3f9b975ec.elb.eu-west-1.amazonaws.com/user_ratings/", {
          //.post("http://localhost:8087/users/" + user_id + "/user_ratings/" + article_id, {
          value: score,
          article_id: article,
          user_id: user
        })
        .then(function(response) {
          console.log(response);
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    get_user_article_rating: function(user, ratings) {
      // console.log("Article: " + article);
      //console.log("User: " + user);
      if (typeof ratings === "undefined") {
        return 0;
      }
      var i;
      for (i = 0; i < ratings.length; i++) {
        // console.log("Ratings " + i + ":" + ratings[i].user_id + "--" + ratings[i].value);
        if (ratings[i].user_id == user) {
          return ratings[i].value;
        }
      }
      return 0;
    },
    user_liked_article: function(user, ratings) {
      return this.get_user_article_rating(user, ratings) == 1;
    },
    user_disliked_article: function(user, ratings) {
      return this.get_user_article_rating(user, ratings) == -1;
    }
  }
};
</script>

<style>
.article.preprint {
  border-left: 4px solid #3cd1c2;
}
</style>