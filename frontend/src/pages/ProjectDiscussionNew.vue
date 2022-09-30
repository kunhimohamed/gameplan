<template>
  <div class="py-6">
    <div class="border p-4 rounded-lg">
      <div class="mb-3 flex items-center space-x-2">
        <UserProfileLink :user="$user().name">
          <Avatar :label="$user().full_name" :imageURL="$user().user_image" />
        </UserProfileLink>
        <div class="flex w-full items-center">
          <div>
            <span class="text-base text-gray-900">
              <UserProfileLink
                class="font-medium hover:text-blue-600"
                :user="$user().name"
              >
                {{ $user().full_name }}
              </UserProfileLink>
              in
              <router-link
                class="hover:text-blue-600"
                :to="{
                  name: 'Team',
                  params: {
                    teamId: project.doc.team,
                  },
                }"
              >
                {{
                  $getDoc('Team', project.doc.team)?.title || project.doc.team
                }}
              </router-link>
              <span class="text-gray-500"> &mdash; </span>
              <router-link
                class="hover:text-blue-600"
                :to="{
                  name: 'ProjectOverview',
                  params: {
                    teamId: project.doc.team,
                    projectId: project.doc.name,
                  },
                }"
              >
                {{ project.doc.title }}
              </router-link>
            </span>
          </div>
        </div>
        <div class="space-x-2 shrink-0 hidden sm:block">
          <Button :route="{ name: 'ProjectDiscussions' }">Discard</Button>
          <Button
            appearance="primary"
            :loading="$resources.newDiscussion.loading"
            @click="$resources.newDiscussion.submit({ title, content })"
          >
            Publish
          </Button>
        </div>
      </div>
      <ErrorMessage :message="$resources.newDiscussion.error" />
      <textarea
        class="w-full px-0 py-0.5 mt-1 text-3xl font-bold border-0 rounded-lg focus:ring-0 placeholder-gray-400 resize-none"
        v-model="title"
        placeholder="Title"
        rows="1"
        wrap="soft"
        maxlength="140"
        v-focus
        @keydown.enter.prevent="$refs.textEditor.editor.commands.focus()"
        @input="
          (e) => {
            e.target.style.height = e.target.scrollHeight + 'px'
          }
        "
      ></textarea>
      <TextEditor
        ref="textEditor"
        class="mt-1"
        editor-class="rounded-b-lg max-w-[unset] prose-sm h-[calc(100vh-410px)] sm:h-[calc(100vh-320px)] overflow-auto"
        :content="content"
        @change="(val) => (content = val)"
        placeholder="Write something..."
      >
        <template v-slot:bottom>
          <div
            class="mt-2 flex flex-col justify-between sm:flex-row sm:items-center"
          >
            <TextEditorFixedMenu
              class="overflow-x-auto"
              :buttons="textEditorMenuButtons"
            />
            <div class="space-x-2 shrink-0 sm:hidden mt-2 text-right">
              <Button :route="{ name: 'ProjectDiscussions' }">Discard</Button>
              <Button
                appearance="primary"
                :loading="$resources.newDiscussion.loading"
                @click="$resources.newDiscussion.submit({ title, content })"
              >
                Publish
              </Button>
            </div>
          </div>
        </template>
      </TextEditor>
    </div>
  </div>
</template>
<script>
import { Avatar } from 'frappe-ui'
import TextEditor from '@/components/TextEditor.vue'
import { focus } from '@/directives'
import UserProfileLink from '@/components/UserProfileLink.vue'
import TextEditorFixedMenu from 'frappe-ui/src/components/TextEditor/TextEditorFixedMenu.vue'

export default {
  name: 'ProjectDiscussionNew',
  props: ['project'],
  components: { TextEditor, Avatar, UserProfileLink, TextEditorFixedMenu },
  directives: { focus },
  data() {
    return {
      title: '',
      content: '',
    }
  },
  resources: {
    newDiscussion() {
      return {
        method: 'frappe.client.insert',
        makeParams({ title, content }) {
          return {
            doc: {
              doctype: 'Team Discussion',
              project: this.project.doc.name,
              title,
              content,
            },
          }
        },
        validate(params) {
          if (!params.doc.title) {
            return `Please enter title before publishing.`
          }
        },
        onSuccess(doc) {
          this.$router.replace({
            name: 'ProjectDiscussion',
            params: {
              teamId: doc.team,
              projectId: doc.project,
              postId: doc.name,
            },
          })
          this.title = ''
          this.content = ''
        },
      }
    },
  },
  computed: {
    textEditorMenuButtons() {
      return [
        'Paragraph',
        ['Heading 2', 'Heading 3', 'Heading 4', 'Heading 5', 'Heading 6'],
        'Separator',
        'Bold',
        'Italic',
        'Separator',
        'Bullet List',
        'Numbered List',
        'Separator',
        'Align Left',
        'Align Center',
        'Align Right',
        'Separator',
        'Image',
        'Link',
        'Blockquote',
        'Code',
        'Horizontal Rule',
        [
          'InsertTable',
          'AddColumnBefore',
          'AddColumnAfter',
          'DeleteColumn',
          'AddRowBefore',
          'AddRowAfter',
          'DeleteRow',
          'MergeCells',
          'SplitCell',
          'ToggleHeaderColumn',
          'ToggleHeaderRow',
          'ToggleHeaderCell',
          'DeleteTable',
        ],
        'Separator',
        'Undo',
        'Redo',
      ]
    },
  },
}
</script>