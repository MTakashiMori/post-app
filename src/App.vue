<template>
    <v-app>
        <v-app-bar app color="primary" dark>
            <div class="d-flex align-center">
                <h3>POST APP</h3>
            </div>

            <v-spacer></v-spacer>
        </v-app-bar>

        <v-main>
            <router-view/>
        </v-main>
    </v-app>
</template>

<script>

    export default {
        name: 'App',
    };
</script>
