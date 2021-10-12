<template>

    <v-card outlined shaped>

        <v-card-title>

            {{item.name}}

            <v-spacer></v-spacer>

            <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                    <v-btn icon v-on="on" color="primary" v-if="commentable" @click="$emit('comment')">
                        <v-icon>mdi-message</v-icon>
                    </v-btn>
                </template>
                <span>Comment</span>
            </v-tooltip>

        </v-card-title>

        <v-card-text>{{ item.comment }}</v-card-text>
    </v-card>

</template>

<script>

    export default {
        name: 'CommentComponent',
        props: [
            'item',
            'commentable'
        ]
    }

</script>