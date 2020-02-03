<template>
  <v-app>
    <v-navigation-drawer
      :clipped="drawer.clipped"
      :fixed="drawer.fixed"
      :mini-variant="drawer.mini"
      v-model="drawer.open"
      app
    >
      <v-list>
        <v-list-tile @click="toggleMiniDrawer" avatar>
          <v-list-tile-avatar>
            <img
              src="https://media.licdn.com/dms/image/C4E03AQFM3ujCogBNJw/profile-displayphoto-shrink_200_200/0?e=1554336000&v=beta&t=uGy0p7yxSCflKlYLYahP6Yse3H2nrmmWQigxlLLxxaE"
            >
          </v-list-tile-avatar>

          <v-list-tile-content>
            <v-list-tile-title>artminda chen</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>

        <v-list-tile href="https://github.com/artminda" target="_blank">
          <v-list-tile-action>
            <v-icon large>fab fa-github</v-icon>
          </v-list-tile-action>

          <v-list-tile-content>
            <v-list-tile-title>Github</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>

      </v-list>
    </v-navigation-drawer>

    <v-toolbar
      app
      :fixed="toolbar.fixed"
      :clipped-left="toolbar.clippedLeft"
      scroll-off-screen
      dark
    >
      <v-toolbar-side-icon @click.stop="toggleDrawer"></v-toolbar-side-icon>
      <v-toolbar-title v-if="!isMobile">Film Locations</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-text-field v-model="search" append-icon="search" label="Search" single-line clearable flat></v-text-field>
      <v-menu
        offset-y
        :nudge-left="50"
        :close-on-content-click="false"
        transition="slide-x-reverse-transition"
      >
        <v-btn icon slot="activator">
          <v-icon>more_vert</v-icon>
        </v-btn>
        <v-list>
          <v-list-tile v-for="(item) in headers" :key="item.value" @click="changeSort(item.value)">
            <v-list-tile-title>
              {{ item.text }}
              <v-icon
                v-if="pagination.sortBy === item.value"
              >{{pagination.descending ? 'arrow_downward':'arrow_upward'}}</v-icon>
            </v-list-tile-title>
          </v-list-tile>
        </v-list>
      </v-menu>
    </v-toolbar>

    <v-content>
      <v-container fluid fill-height>
        <v-layout justify-center>
          <v-flex shrink>
            <v-layout v-resize="onResize" column style="padding-top:56px">
              <v-data-table
                :headers="headers"
                :items="movies"
                :expand="expand"
                :search="search"
                :pagination.sync="pagination"
                :hide-headers="isMobile"
                :class="{mobile: isMobile}"
                dark
                item-key="title"
              >
                <template slot="items" slot-scope="props">
                  <tr v-if="!isMobile" @click="props.expanded = !props.expanded">
                    <td>{{ props.item.title }}</td>
                    <td>{{ props.item.release_year }}</td>
                    <td>{{ props.item.director }}</td>
                  </tr>
                  <tr v-else @click="dialog = true; props.expanded = !props.expanded">
                    <td>
                      <ul class="flex-content">
                        <li class="flex-item" data-label="Title">{{ props.item.title }}</li>
                        <li class="flex-item" data-label="Year">{{ props.item.release_year }}</li>
                        <li class="flex-item" data-label="Director">{{ props.item.director }}</li>
                      </ul>
                    </td>
                  </tr>
                </template>
                <template slot="expand" slot-scope="props">
                  <v-card v-if="!isMobile" flat light>
                    <v-card-text>
                      <div class="font-weight-bold">Location
                        <v-icon small>fas fa-map-marker-alt</v-icon>
                      </div>
                      {{props.item.locations}}
                      <div class="font-weight-bold">Actors
                        <v-icon small>fas fa-users</v-icon>
                      </div>
                      {{props.item.actor_1}}, {{props.item.actor_2}}
                      <div class="font-weight-bold">Writer
                        <v-icon small>fas fa-user-edit</v-icon>
                      </div>
                      {{props.item.writer}}
                    </v-card-text>
                  </v-card>
                  <v-dialog v-else v-model="dialog" max-width="290">
                    <v-card>
                      <v-card-text>
                        <div class="font-weight-bold">Location
                          <v-icon small>fas fa-map-marker-alt</v-icon>
                        </div>
                        {{props.item.locations}}
                        <div class="font-weight-bold">Actors
                          <v-icon small>fas fa-users</v-icon>
                        </div>
                        {{props.item.actor_1}}, {{props.item.actor_2}}
                        <div class="font-weight-bold">Writer
                          <v-icon small>fas fa-user-edit</v-icon>
                        </div>
                        {{props.item.writer}}
                      </v-card-text>
                    </v-card>
                  </v-dialog>
                </template>
                <v-alert
                  outline
                  slot="no-results"
                  :value="true"
                  color="error"
                  icon="warning"
                >Your search for "{{ search }}" found no results.</v-alert>
              </v-data-table>
            </v-layout>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>

    <v-footer app :fixed="footer.fixed" :clipped-left="footer.clippedLeft" dark>
      <span class="caption mx-3">&copy; 2019, artminda chen</span>
      <v-spacer></v-spacer>
      <span class="caption mx-3">lodash practice</span>
    </v-footer>
  </v-app>
</template>

<script>
import axios from "axios";
import _ from "lodash";

export default {
  name: "App",
  data() {
    return {
      expand: false,
      dialog: false,
      pagination: {
        sortBy: "name"
      },
      search: "",
      isMobile: false,
      headers: [
        {
          text: "Title",
          align: "left",
          value: "title"
        },
        {
          text: "Year",
          value: "release_year"
        },
        {
          text: "Director",
          value: "director"
        }
      ],
      movies: [],
      drawer: {
        // sets the open status of the drawer
        open: false,
        // sets if the drawer is shown above (false) or below (true) the toolbar
        clipped: true,
        // sets if the drawer is CSS positioned as 'fixed'
        fixed: false,
        // sets if the drawer remains visible all the time (true) or not (false)
        permanent: true,
        // sets the drawer to the mini variant, showing only icons, of itself (true)
        // or showing the full drawer (false)
        mini: true
      },
      toolbar: {
        //
        fixed: true,
        // sets if the toolbar contents is leaving space for drawer (false) or not (true)
        clippedLeft: true
      },
      footer: {
        // sets the CSS position of the footer
        fixed: true,
        // sets if the footer is full width (true) or gives space to the drawer (false)
        clippedLeft: true
      }
    };
  },

  created() {
    axios
      .get(`https://data.sfgov.org/resource/wwmu-gmzc.json`)
      .then(response => {
        this.movies = _.uniqBy(response.data, "title");
        console.log(this.movies);
      })
      .catch(e => {
        this.errors.push(e);
      });
  },

  methods: {
    onResize() {
      if (window.innerWidth < 769) this.isMobile = true;
      else this.isMobile = false;
    },

    changeSort(column) {
      console.log(column);
      if (this.pagination.sortBy === column) {
        this.pagination.descending = !this.pagination.descending;
      } else {
        this.pagination.sortBy = column;
        this.pagination.descending = false;
      }
    },
    // toggles the drawer variant (mini/full)
    toggleMiniDrawer() {
      this.drawer.mini = !this.drawer.mini;
    },
    // toggles the drawer type (permanent vs temporary) or shows/hides the drawer
    toggleDrawer() {
      this.drawer.permanent = false;
      if (this.drawer.permanent) {
        this.drawer.permanent = !this.drawer.permanent;
        // set the clipped state of the drawer and toolbar
        this.drawer.clipped = true;
        this.toolbar.clippedLeft = true;
      } else {
        // normal drawer
        this.drawer.open = !this.drawer.open;
      }
    }
  }
};
</script>


<style>
.mobile {
  color: #333;
}

@media screen and (max-width: 768px) {
  .mobile table.v-table tr {
    max-width: 100%;
    position: relative;
    display: block;
  }

  .mobile table.v-table tr td {
    display: flex;
    width: 100%;
    border-bottom: 1px solid #f5f5f5;
    height: auto;
    padding: 10px;
  }

  .mobile table.v-table tr td ul li:before {
    content: attr(data-label);
    padding-right: 0.5em;
    text-align: left;
    display: block;
    color: #999;
  }
  .v-datatable__actions__select {
    width: 50%;
    margin: 0px;
    justify-content: flex-start;
  }
  .mobile .theme--light.v-table tbody tr:hover:not(.v-datatable__expand-row) {
    background: transparent;
  }
}
.flex-content {
  padding: 0;
  margin: 0;
  list-style: none;
  display: flex;
  flex-wrap: wrap;
  width: 100%;
}

.flex-item {
  padding: 5px;
  width: 50%;
  height: 40px;
  font-weight: bold;
}
</style>