<template>

    <v-container style="margin-top: -10%">

        <v-tooltip top>
            <template v-slot:activator="{ on }">
                <v-fab-transition>
                    <v-btn left fab large absolute bottom right class="add-post primary" v-on="on" @click="addPost">
                        <v-icon xl>mdi-plus</v-icon>
                    </v-btn>
                </v-fab-transition>
            </template>
            <span>Add post</span>
        </v-tooltip>

        <v-card v-for="item in data" :key="item.id" style="margin-top: 5%">
            <v-card-title>

                {{item.name}}

                <v-spacer></v-spacer>

                <v-tooltip bottom>
                    <template v-slot:activator="{ on }">
                        <v-btn icon v-on="on" color="primary" @click="commentItem(item, 'Post')">
                            <v-icon>mdi-message</v-icon>
                        </v-btn>
                    </template>
                    <span>Comment</span>
                </v-tooltip>

            </v-card-title>

            <v-divider></v-divider>

            <v-card-text v-if="item.comments.length > 0">

                <v-timeline dense>

                    <v-timeline-item v-for="comment in item.comments" :key="comment.id" hide-dot>

                        <comment-component :item="comment" :commentable="true" @comment="commentItem(comment, 'Comment')"></comment-component>

                        <v-timeline dense v-if="comment.comments.length > 0 ">

                            <v-timeline-item v-for="subcomment in comment.comments" :key="subcomment.id" hide-dot>

                                <comment-component :item="subcomment" :commentable="true" @comment="commentItem(subcomment, 'Comment')"></comment-component>

                                <v-timeline dense v-if="subcomment.comments.length > 0 ">

                                    <v-timeline-item v-for="subsubcomment in subcomment.comments" :key="subsubcomment.id" hide-dot>

                                        <comment-component :item="subsubcomment" :commentable="false" @comment="commentItem(subsubcomment, 'Comment')"></comment-component>

                                    </v-timeline-item>

                                </v-timeline>

                            </v-timeline-item>

                        </v-timeline>

                    </v-timeline-item>

                </v-timeline>

            </v-card-text>

        </v-card>


        <v-dialog v-model="commentDialog.status" width="50%" persistent>

            <v-card>
                <v-card-title>
                    Write your comment above!
                </v-card-title>

                <v-card-text>

                    <v-form ref="commentForm">

                        <v-text-field label="Name" :rules="[ v => !!v || 'Name is required' ]" v-model="commentDialog.name"></v-text-field>

                        <v-textarea label="Comment" :rules="[ v => !!v || 'Comment is required' ]" v-model="commentDialog.comment"></v-textarea>

                    </v-form>

                </v-card-text>

                <v-divider></v-divider>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-tooltip bottom>
                        <template v-slot:activator="{ on }">
                            <v-btn text v-on="on" color="primary" @click="submitComment(false)">
                                Cancel
                            </v-btn>
                        </template>
                        <span>Cancel comment</span>
                    </v-tooltip>

                    <v-tooltip bottom>
                        <template v-slot:activator="{ on }">
                            <v-btn v-on="on" class="primary" @click="submitComment(true)">
                                Save
                            </v-btn>
                        </template>
                        <span>Save comment</span>
                    </v-tooltip>

                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="postDialog.status" width="50%" persistent>

            <v-card>
                <v-card-title>
                    Write a post
                </v-card-title>

                <v-card-text>

                    <v-form ref="postForm">

                        <v-text-field label="Name" :rules="[ v => !!v || 'Name is required' ]" v-model="postDialog.name"></v-text-field>

                    </v-form>

                </v-card-text>

                <v-divider></v-divider>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-tooltip bottom>
                        <template v-slot:activator="{ on }">
                            <v-btn text v-on="on" color="primary" @click="submitPost(false)">
                                Cancel
                            </v-btn>
                        </template>
                        <span>Cancel comment</span>
                    </v-tooltip>

                    <v-tooltip bottom>
                        <template v-slot:activator="{ on }">
                            <v-btn v-on="on" class="primary" @click="submitPost(true)">
                                Save
                            </v-btn>
                        </template>
                        <span>Save comment</span>
                    </v-tooltip>

                </v-card-actions>
            </v-card>
        </v-dialog>


    </v-container>

</template>

<script>

    import axios from 'axios';
    import CommentComponent from "@/views/CommentComponent.vue";

    export default {
        name: 'Home',
        components: {
            CommentComponent
        },
        data() {
            return {
                data: [],
                commentDialog: {
                    status: false,
                    id: '',
                    type: '',
                    name: '',
                    comment: ''
                },
                postDialog: {
                    status: false,
                    name: '',
                }
            }
        },
        mounted() {
            this.getAll();
        },
        methods: {
            getAll() {
                axios.get(process.env.VUE_APP_API_URL + '/post' + '?orderDesc=true&sortBy=created_at')
                    .then((res) => {
                        this.data = res.data.data.data;
                    });
            },
            commentItem(item, type) {
                this.commentDialog.status = true;
                this.commentDialog.id = item.id;
                this.commentDialog.type = type;
            },
            submitComment(response) {
                if(!response) {
                    this.commentDialog.status = false;
                    this.cleanDialog();
                    return;
                }

                if(!this.$refs.commentForm.validate()) {
                    return;
                }

                axios.post(process.env.VUE_APP_API_URL + '/comment', {
                    commentable_id: this.commentDialog.id,
                    commentable_type: "App\\Models\\" + this.commentDialog.type,
                    comment: this.commentDialog.comment,
                    name: this.commentDialog.name,
                }).then(() => {
                    this.getAll();
                    this.cleanDialog();
                });
            },
            cleanDialog() {
                this.commentDialog.status = false;
                this.commentDialog.id = '';
                this.commentDialog.type = '';
                this.commentDialog.name = '';
                this.commentDialog.comment = '';
                this.$refs.commentForm.reset()
            },
            cleanPostDialog() {
                this.postDialog.status = false;
                this.postDialog.name = '';
                this.$refs.postForm.reset()
            },
            addPost() {
                this.postDialog.status = true;
            },
            submitPost(response) {
                if(!response) {
                    this.postDialog.status = false;
                    this.cleanPostDialog();
                    return;
                }

                if(!this.$refs.postForm.validate()) {
                    return;
                }

                axios.post(process.env.VUE_APP_API_URL + '/post', {
                    name: this.postDialog.name,
                }).then(() => {
                    this.getAll();
                    this.cleanPostDialog();
                });
            }
        }
    }

</script>

<style>

    .add-post {
        position: absolute;
        position: -webkit-sticky;
        position: sticky;
        top: 90%;
        margin-left: 100%;
        z-index: 999;
    }

</style>
