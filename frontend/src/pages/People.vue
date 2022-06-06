<template>
  <div class="flex flex-col h-full">
    <header class="flex items-center h-12 px-4 py-3 bg-white border-b">
      <Breadcrumbs
        :breadcrumbs="[
          {
            title: 'People',
            icon: '👨‍👩‍👧‍👦',
            route: { name: 'People' },
          },
          person && {
            title: $user(person).full_name,
            icon: '',
          },
        ]"
      />
    </header>
    <div class="flex flex-1 min-h-0">
      <div class="w-1/3 overflow-auto">
        <div class="py-6 pl-6 space-y-2">
          <div v-for="user in $resources.users.data" :key="user.name">
            <router-link
              :to="{ name: 'People', params: { person: user.name } }"
              class="flex items-center w-full p-4 border rounded-xl hover:bg-gray-50"
              exact-active-class="!bg-gray-100"
            >
              <Avatar :label="user.full_name" :imageURL="user.user_image" />
              <div class="ml-2">
                <div class="text-lg text-gray-900">{{ user.full_name }}</div>
                <div class="text-base text-gray-600">{{ user.email }}</div>
              </div>
              <Badge
                class="ml-auto"
                :color="{
                  green: user.status == 'Available',
                  yellow: user.status == 'Away',
                  red: user.status == 'Busy',
                }"
              >
                {{ user.status }}
              </Badge>
            </router-link>
          </div>
        </div>
      </div>
      <div class="w-2/3 overflow-auto">
        <PeopleProfile v-if="person" :user="person" />
      </div>
    </div>
  </div>
</template>
<script>
import { Avatar, Badge } from 'frappe-ui'
import Breadcrumbs from '@/components/Breadcrumbs.vue'
import PeopleProfile from './PeopleProfile.vue'

export default {
  name: 'People',
  props: ['person'],
  components: { Breadcrumbs, Avatar, PeopleProfile, Badge },
  resources: {
    users() {
      return {
        type: 'list',
        doctype: 'Team User Profile',
        cache: 'People',
        fields: [
          'name',
          'user.full_name',
          'user.email',
          'user.user_image',
          'status',
        ],
      }
    },
  },
}
</script>