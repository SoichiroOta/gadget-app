<template>
  <v-container>
    <ErrorCard
      :display="error"
      title="404NotFount"
      message="掲示板が存在しません。"
    />
    <CreateBoardCommentDialog
      :dialog="createCommentDialog"
      :boardId="boardId"
      @createBoardComment="createBoardComment"
      @closeDialog="createCommentDialog = false"
    />
    <CreateBoardReplyDialog
      :dialog="createReplyDialog"
      :boardId="boardId"
      :parentComment="parentComment"
      @createBoardReply="createBoardReply"
      @closeDialog="createReplyDialog = false"
    />
    <DeleteBoardCommentDialog
      :dialog="deleteCommentDialog"
      :boardId="boardId"
      :comment="comment"
      :deletemode="deletemode"
      @deleteBoardComment="deleteBoardComment"
      @closeDialog="deleteCommentDialog = false"
    />
    <EditBoardDialog
      :dialog="editBoardDialog"
      :boardId="boardId"
      @editBoard="editBoard"
      @closeDialog="editBoardDialog = false"
    />
    <DeleteBoardDialog
      :dialog="deleteBoardDialog"
      :boardId="boardId"
      :deletemode="deletemode"
      @deleteBoard="deleteBoard"
      @closeDialog="deleteBoardDialog = false"
    />
    <v-row
      justify="center"
      v-if="error === false"
    >
      <v-col lg="6" sm="8" cols="12">
        <v-card>
          <v-toolbar
            class="cyan darken-1"
            flat
          >
            <v-icon
              dark
              v-if="board.board_type === '雑談'">
              mdi-forum
            </v-icon>
            <v-icon
              dark
              v-if="board.board_type === '質問'">
              mdi-comment-question
            </v-icon>
            <v-toolbar-title
              class="white--text font-weight-bold ml-3"
            >
              {{ board.title }}
            </v-toolbar-title>
          </v-toolbar>
          <v-container>
            <v-card
              class="mx-auto"
              flat
            >
              <v-card-title>
                <v-container>
                  <v-row>
                    <v-col cols="4">
                      <v-avatar 
                      size="64"
                      >
                        <img 
                          v-if="board.user.avatar_url"
                          :src="board.user.avatar_url"
                          alt="Avatar"
                        >
                        <img
                          v-else
                          src="~/assets/images/default_icon.jpeg"
                          alt="Avatar"
                        >
                      </v-avatar>
                    </v-col>
                    <v-col cols="8">
                      <v-card flat :to="`/users/${board.user.id}`">
                        <h3>{{ board.user.name }}</h3>
                        <h6>{{ $moment(board.created_at).format('YYYY年MM月DD日 HH時mm分') }}</h6>
                      </v-card>
                    </v-col>
                    <v-col cols="12">
                      <Tags
                        :tags="board.tags"
                      />
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-card
                        flat
                        :to="`/boards/${board.id}`"
                      >
                        <span>{{ board.description }}</span>
                      </v-card>
                    </v-col>
                    <v-col cols="12" v-if="board.images_url">
                      <Images
                        :images="board.images_url"
                      />
                    </v-col>
                    <v-col cols="12" v-if="$store.state.modules.user.data">
                      <v-row justify="end">
                        <!---------------->
                        <!-- 管理メニュー -->
                        <!---------------->
                        <div v-if="$store.state.modules.user.data">
                          <!-- adminユーザーの場合 -->
                          <div v-if="$store.state.modules.user.data.admin === true">
                            <!-- 自身が作成したコンテンツの場合 -->
                            <div v-if="$store.state.modules.user.data.id === board.user.id">
                              <v-btn
                                icon
                                text
                                color="grey darken-2"
                                @click="openEditBoardDialog"
                              >
                                <v-icon>
                                  mdi-pencil-box-multiple
                                </v-icon>
                              </v-btn>
                              <v-btn
                                icon
                                text
                                color="grey darken-2"
                                @click="openDeleteBoardDialog(mode.owner)"
                              >
                                <v-icon>
                                  mdi-delete
                                </v-icon>
                              </v-btn>
                            </div>
                            <div v-else>
                              <v-btn
                                icon
                                text
                                color="grey darken-2"
                                @click="openDeleteBoardDialog(mode.admin)"
                              >
                                <v-icon>
                                  mdi-delete
                                </v-icon>
                              </v-btn>
                            </div>
                          </div>
                          <!-- 一般ユーザーの場合 -->
                          <div v-if="$store.state.modules.user.data.admin === false">
                            <!-- 自身が作成したコンテンツの場合 -->
                            <div v-if="$store.state.modules.user.data.id === board.user.id">
                              <v-btn
                                icon
                                text
                                color="grey darken-2"
                                @click="openEditBoardDialog"
                              >
                                <v-icon>
                                  mdi-pencil-box-multiple
                                </v-icon>
                              </v-btn>
                              <v-btn
                                icon
                                text
                                color="grey darken-2"
                                @click="openDeleteBoardDialog(mode.owner)"
                              >
                                <v-icon>
                                  mdi-delete
                                </v-icon>
                              </v-btn>
                            </div>
                          </div>
                        </div>
                      </v-row>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card flat>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-btn
                        block
                        color="orange"
                        dark
                        rounded
                        @click="openCreateCommentDialog"
                      >
                        コメントする
                      </v-btn>
                    </v-col>
                    <v-col
                      cols="12"
                      v-for="comment in board.board_comments"
                      :key="comment.id"
                    >
                      <v-card
                        class="mx-auto pa-3"
                      >
                        <v-container>
                          <v-row>
                            <v-col>
                              <v-avatar 
                                size="64"
                              >
                                <img 
                                  v-if="comment.user.avatar_url"
                                  :src="comment.user.avatar_url"
                                  alt="Avatar"
                                >
                                <img
                                  v-else
                                  src="~/assets/images/default_icon.jpeg"
                                  alt="Avatar"
                                >
                              </v-avatar>
                            </v-col>
                            <v-col align-self="center">
                              <v-card flat :to="`/users/${comment.user.id}`">
                                <h3>{{ comment.user.name }}</h3>
                                <h6>{{ $moment(comment.created_at).format('YYYY年MM月DD日 HH時mm分') }}</h6>
                              </v-card>
                            </v-col>
                          </v-row>
                          <Images
                            :images="comment.images_url"
                          />
                          <v-row>
                            <v-col>
                              <span>{{ comment.description }}</span>
                            </v-col>
                          </v-row>
                          <v-row justify="end">
                            <!---------------->
                            <!-- 管理メニュー -->
                            <!---------------->
                            <div v-if="$store.state.modules.user.data">
                              <!-- adminユーザーの場合 -->
                              <div v-if="$store.state.modules.user.data.admin === true">
                                <!-- 自身が作成したコンテンツの場合 -->
                                <div v-if="$store.state.modules.user.data.id === comment.user.id">
                                  <v-btn
                                    icon
                                    text
                                    color="grey darken-2"
                                    @click="openDeleteCommentDialog(comment, mode.owner)"
                                  >
                                    <v-icon>
                                      mdi-delete
                                    </v-icon>
                                  </v-btn>
                                </div>
                                <div v-else>
                                  <v-btn
                                    icon
                                    text
                                    color="grey darken-2"
                                    @click="openDeleteCommentDialog(comment, mode.admin)"
                                  >
                                    <v-icon>
                                      mdi-delete
                                    </v-icon>
                                  </v-btn>
                                </div>
                              </div>
                              <!-- 一般ユーザーの場合 -->
                              <div v-if="$store.state.modules.user.data.admin === false">
                                <!-- 自身が作成したコンテンツの場合 -->
                                <div v-if="$store.state.modules.user.data.id === comment.user.id">
                                  <v-btn
                                    icon
                                    text
                                    color="grey darken-2"
                                    @click="openDeleteCommentDialog(comment, mode.owner)"
                                  >
                                    <v-icon>
                                      mdi-delete
                                    </v-icon>
                                  </v-btn>
                                </div>
                              </div>
                            </div>
                            <v-btn
                              rounded
                              color="success"
                              class="cyan darken-1"
                              @click="openCreateReplyDialog(comment)"
                            >
                              返信
                            </v-btn>
                          </v-row>
                        </v-container>
                      </v-card>
                      <v-timeline
                        v-if="'childComments' in comment"
                        align-top
                      >
                        <v-timeline-item
                          v-for="child in comment.childComments"
                          :key="child.id"
                          small
                          color="grey"
                          right
                          hide-dot
                        >
                          <v-card
                            color="grey"
                          >
                            <v-card-title class="title white--text">
                              <v-icon dark>mdi-reply</v-icon>
                              返信コメント
                            </v-card-title>
                            <v-card-text class="white text--primary">
                              <v-container>
                                <v-row>
                                  <v-col>
                                    <v-avatar 
                                      size="48"
                                    >
                                      <img 
                                        v-if="child.user.avatar_url"
                                        :src="child.user.avatar_url"
                                        alt="Avatar"
                                      >
                                      <img
                                        v-else
                                        src="~/assets/images/default_icon.jpeg"
                                        alt="Avatar"
                                      >
                                    </v-avatar>
                                  </v-col>
                                  <v-col align-self="center">
                                    <v-card flat :to="`/users/${child.user.id}`">
                                      <h3>{{ child.user.name }}</h3>
                                      <h6>{{ $moment(child.created_at).format('YYYY年MM月DD日 HH時mm分') }}</h6>
                                    </v-card>
                                  </v-col>
                                </v-row>
                                <v-row>
                                  <v-col>
                                    <span>{{ child.description }}</span>
                                  </v-col>
                                </v-row>
                                <Images
                                  :images="child.images_url"
                                />
                                <v-row
                                  justify="end"
                                >
                                  <!---------------->
                                  <!-- 管理メニュー -->
                                  <!---------------->
                                  <div v-if="$store.state.modules.user.data">
                                    <!-- adminユーザーの場合 -->
                                    <div v-if="$store.state.modules.user.data.admin === true">
                                      <!-- 自身が作成したコンテンツの場合 -->
                                      <div v-if="$store.state.modules.user.data.id === child.user.id">
                                        <v-btn
                                          icon
                                          text
                                          color="grey darken-2"
                                          @click="openDeleteCommentDialog(child, mode.owner)"
                                        >
                                          <v-icon>
                                            mdi-delete
                                          </v-icon>
                                        </v-btn>
                                      </div>
                                      <div v-else>
                                        <v-btn
                                          icon
                                          text
                                          color="grey darken-2"
                                          @click="openDeleteCommentDialog(child, mode.admin)"
                                        >
                                          <v-icon>
                                            mdi-delete
                                          </v-icon>
                                        </v-btn>
                                      </div>
                                    </div>
                                    <!-- 一般ユーザーの場合 -->
                                    <div v-if="$store.state.modules.user.data.admin === false">
                                      <!-- 自身が作成したコンテンツの場合 -->
                                      <div v-if="$store.state.modules.user.data.id === child.user.id">
                                        <v-btn
                                          icon
                                          text
                                          color="grey darken-2"
                                          @click="openDeleteCommentDialog(child, mode.owner)"
                                        >
                                          <v-icon>
                                            mdi-delete
                                          </v-icon>
                                        </v-btn>
                                      </div>
                                    </div>
                                  </div>
                                </v-row>
                              </v-container>
                            </v-card-text>
                          </v-card>
                        </v-timeline-item>
                      </v-timeline>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card>
            </v-card>
          </v-container>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import { mapGetters, mapActions } from 'vuex'
import ErrorCard from '~/components/molecules/ErrorCard.vue'
import Tags from "~/components/atoms/Tags.vue"
import Images from "~/components/atoms/Images.vue"
import CreateBoardCommentDialog from '~/components/organisms/boards/CreateBoardCommentDialog.vue'
import CreateBoardReplyDialog from '~/components/organisms/boards/CreateBoardReplyDialog.vue'
import DeleteBoardCommentDialog from '~/components/organisms/boards/DeleteBoardCommentDialog.vue'
import EditBoardDialog from '~/components/organisms/boards/EditBoardDialog.vue'
import DeleteBoardDialog from '~/components/organisms/boards/DeleteBoardDialog.vue'
export default {
  components: {
    ErrorCard,
    Tags,
    Images,
    CreateBoardCommentDialog,
    CreateBoardReplyDialog,
    DeleteBoardCommentDialog,
    EditBoardDialog,
    DeleteBoardDialog
  },
  data () {
    return {
      boardId: '',
      createCommentDialog: false,
      createReplyDialog: false,
      deleteCommentDialog: false,
      editBoardDialog: false,
      deleteBoardDialog: false,
      parentComment: '',
      comment: '',
      mode: {
        owner: 'owner',
        admin: 'admin'
      },
      deletemode: ''
    }
  },
  async fetch ({ $axios, params, store }) {
    const baseUrl = process.client ? process.env.BROWSER_BASE_URL : process.env.API_BASE_URL
    await $axios.$get(baseUrl + `/v1/boards/${params.id}`)
      .then(res => {
        // 掲示板の情報をコミット
        store.dispatch('modules/board/setData', res)
        store.commit('modules/info/setError', false)
      })
      .catch(error => {
        store.commit('modules/info/setError', true)
      })
  },


  computed: {
    ...mapGetters({
      currentUser: 'modules/user/data',
      isAuthenticated: 'modules/user/isAuthenticated',
      board: 'modules/board/data',
      error: 'modules/info/error'
    })
  },

  methods: {
    ...mapActions({
      reloadBoardByEditBoard: 'modules/board/reloadBoardByEditBoard',
      reloadBoardByCreateComment: 'modules/board/reloadBoardByCreateComment',
      reloadBoardByDeleteComment: 'modules/board/reloadBoardByDeleteComment'
    }),
    openCreateCommentDialog () {
      this.boardId = this.board.id
      this.createCommentDialog = true
    },
    openCreateReplyDialog (comment) {
      this.boardId = this.board.id
      this.parentComment = comment
      this.createReplyDialog = true
    },
    openDeleteCommentDialog (comment, payload) {
      this.deletemode = payload
      this.boardId = this.board.id
      this.comment = comment
      this.deleteCommentDialog = true
    },
    openEditBoardDialog () {
      this.boardId = this.board.id
      this.editBoardDialog = true
    },
    openDeleteBoardDialog (payload) {
      this.deletemode = payload
      this.boardId = this.board.id
      this.deleteBoardDialog = true
    },
    editBoard (payload) {
      this.reloadBoardByEditBoard(payload)
    },
    deleteBoard () {
      this.$router.push("/")
    },
    createBoardComment (payload) {
      this.reloadBoardByCreateComment(payload)
    },
    createBoardReply (payload) {
      this.reloadBoardByCreateComment(payload)
    },
    deleteBoardComment (payload) {
      this.reloadBoardByDeleteComment(payload)
    }
  }
}
</script>
<style scoped>

</style>