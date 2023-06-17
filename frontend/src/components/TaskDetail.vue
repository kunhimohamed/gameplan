<template>
  <div class="flex h-full flex-1" v-if="$resources.task.doc">
    <div class="w-full flex-1">
      <div class="relative p-6">
        <div
          class="absolute right-0 top-0 p-6"
          v-show="$resources.task.setValueDebounced.loading"
        >
          <LoadingText
            v-if="!$resources.task.setValueDebounced.error"
            text="Saving..."
          />
          <ErrorMessage :message="$resources.task.setValueDebounced.error" />
        </div>
        <input
          type="text"
          placeholder="Title"
          class="mb-2 w-full rounded-md border-none p-0 text-2xl font-semibold text-gray-900 focus:outline-none focus:ring-0"
          @change="
            $resources.task.setValueDebounced.submit({
              title: $event.target.value,
            })
          "
          v-model="$resources.task.doc.title"
          v-focus
        />
        <TextEditor
          ref="description"
          editor-class="prose-sm max-w-none"
          placeholder="Description"
          :content="$resources.task.doc.description"
          :bubbleMenu="true"
          :floating-menu="true"
        />
        <div class="mt-8 flex flex-wrap items-center gap-2 sm:hidden">
          <Autocomplete
            placeholder="Assign a user"
            :options="assignableUsers"
            :value="$resources.task.doc.assigned_to"
            @change="
              (option) => {
                $resources.task.setValue.submit({
                  assigned_to: option?.value || '',
                })
              }
            "
          />
          <TextInput
            type="date"
            placeholder="Due date"
            v-model="$resources.task.doc.due_date"
            @change="
              $resources.task.setValue.submit({
                due_date: $event.target.value,
              })
            "
          />
          <Dropdown :options="statusOptions">
            <Button>
              <template #prefix>
                <TaskStatusIcon :status="$resources.task.doc.status" />
              </template>
              {{ $resources.task.doc.status || 'Set status' }}
            </Button>
          </Dropdown>
          <Dropdown :options="priorityOptions">
            <Button>
              <template v-if="$resources.task.doc.priority" #prefix>
                <TaskPriorityIcon :priority="$resources.task.doc.priority" />
              </template>
              {{ $resources.task.doc.priority || 'Set priority' }}
            </Button>
          </Dropdown>
        </div>
        <CommentsList class="mt-8" doctype="GP Task" :name="taskId" />
      </div>
    </div>
    <div class="hidden w-[20rem] shrink-0 border-l sm:block">
      <div
        class="grid grid-cols-2 items-center gap-y-6 p-6 text-base text-gray-700"
      >
        <div>Assignee</div>
        <div>
          <Autocomplete
            placeholder="Assign a user"
            :options="assignableUsers"
            :value="$resources.task.doc.assigned_to"
            @change="
              (option) => {
                $resources.task.setValue.submit({
                  assigned_to: option?.value || '',
                })
              }
            "
          />
        </div>
        <div>Due Date</div>
        <div>
          <TextInput
            type="date"
            placeholder="Due date"
            v-model="$resources.task.doc.due_date"
            @change="
              $resources.task.setValue.submit({
                due_date: $event.target.value,
              })
            "
          />
        </div>
        <div>Status</div>
        <div>
          <Dropdown :options="statusOptions">
            <Button>
              <template #prefix>
                <TaskStatusIcon :status="$resources.task.doc.status" />
              </template>
              {{ $resources.task.doc.status || 'Set status' }}
            </Button>
          </Dropdown>
        </div>
        <div>Priority</div>
        <div>
          <Dropdown :options="priorityOptions">
            <Button>
              <template v-if="$resources.task.doc.priority" #prefix>
                <TaskPriorityIcon :priority="$resources.task.doc.priority" />
              </template>
              {{ $resources.task.doc.priority || 'Set priority' }}
            </Button>
          </Dropdown>
        </div>
      </div>
    </div>
    <teleport to="#home-actions">
      <Dropdown
        :options="[
          {
            label: 'Delete',
            onClick: () => {
              $dialog({
                title: 'Delete task',
                message: 'Are you sure you want to delete this task?',
                actions: [
                  {
                    label: 'Delete',
                    theme: 'red',
                    variant: 'solid',
                    onClick({ close }) {
                      return $resources.task.delete.submit(null, {
                        onSuccess() {
                          close()
                          $router.back()
                        },
                      })
                    },
                  },
                ],
              })
            },
          },
        ]"
      >
        <Button variant="ghost">
          <template #icon><LucideMoreHorizontal class="h-4 w-4" /></template>
        </Button>
      </Dropdown>
    </teleport>
  </div>
</template>
<script>
import { h } from 'vue'
import TextEditor from '@/components/TextEditor.vue'
import ReadmeEditor from '@/components/ReadmeEditor.vue'
import CommentsArea from '@/components/CommentsArea.vue'
import { focus } from '@/directives'
import { Autocomplete, Dropdown, LoadingText, TextInput } from 'frappe-ui'
import CommentsList from '@/components/CommentsList.vue'
import TaskStatusIcon from '@/components/icons/TaskStatusIcon.vue'
import TaskPriorityIcon from '@/components/icons/TaskPriorityIcon.vue'

export default {
  name: 'TaskDetail',
  props: ['taskId'],
  directives: { focus },
  resources: {
    task() {
      return {
        type: 'document',
        doctype: 'GP Task',
        name: this.taskId,
        whitelistedMethods: {
          trackVisit: 'track_visit',
        },
        setValue: {
          onError(e) {
            let message = e.messages ? e.messages.join('\n') : e.message
            this.$toast({
              title: 'Task Update Error',
              text: message,
              icon: 'alert-circle',
              iconClasses: 'text-red-600',
            })
          },
        },
        onSuccess(doc) {
          if (
            ['ProjectTaskDetail', 'HomeTask'].includes(this.$route.name) &&
            Number(this.$route.params.taskId) === doc.name
          ) {
            this.$resources.task.trackVisit.submit()
          }
          this.setupEditorBlur()
        },
      }
    },
  },
  computed: {
    assignableUsers() {
      return this.$users.data
        .filter((user) => user.name != this.$resources.task.doc.assigned_to)
        .map((user) => ({
          label: user.full_name,
          value: user.name,
        }))
    },
    statusOptions() {
      return ['Backlog', 'Todo', 'In Progress', 'Done', 'Canceled'].map(
        (status) => {
          return {
            icon: () => h(TaskStatusIcon, { status }),
            label: status,
            onClick: () => this.$resources.task.setValue.submit({ status }),
          }
        }
      )
    },
    priorityOptions() {
      return ['Low', 'Medium', 'High'].map((priority) => {
        return {
          icon: () => h(TaskPriorityIcon, { priority }),
          label: priority,
          onClick: () => this.$resources.task.setValue.submit({ priority }),
        }
      })
    },
  },
  methods: {
    setupEditorBlur() {
      if (this._blurSetup) return
      let editor = this.$refs.description.editor
      if (!editor) {
        console.log('editor not ready, trying again in 100ms')
        setTimeout(() => this.setupEditorBlur(), 100)
        return
      }
      editor.on('blur', () => {
        this.$resources.task.setValueDebounced.submit({
          description: editor.getHTML(),
        })
      })
      this._blurSetup = true
    },
  },
  components: {
    ReadmeEditor,
    TextEditor,
    CommentsArea,
    Autocomplete,
    TextInput,
    Dropdown,
    CommentsList,
    TaskStatusIcon,
    LoadingText,
    TaskPriorityIcon,
  },
}
</script>