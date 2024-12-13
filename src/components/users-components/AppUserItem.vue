<template>
  <div @click="routToUserProfile" ref="userConteiner" class="creator__container">
    <div class="data__wrapper">
      <div class="creator__avatar">
        <img :src="NO_AVATAR" alt="ava_img">
      </div>
      <div class="creator__name">{{ user.username }}</div>
    </div>
    <div v-if="!isCurrentUserItem" class="buttons__wrapper">
      <my-button v-if="!userIsFavorite?.isLiked" @click.stop="followUser" font-size="14" :title="translate('BTNS.FOLLOW')"/>
      <my-button v-else @click.stop="unfollowUser" background-color="var(--app-pressed-btn)" font-size="14" :title="translate('BTNS.UNFOLLOW')"/>
    </div>
  </div>
</template>

<script setup lang="ts">
import { defineProps, defineEmits, onMounted, ref } from 'vue';
import { NO_AVATAR } from '@/constants';
import { requestService, routingService } from '@/services';
import { CurrentUser, ExistsInFavoritesResponse, User } from '@/models';
import { useAppI18n } from '@/i18n'; 
import { useUserStore } from '@/stores';
const { translate } = useAppI18n()

const { getCurrentUserData } = useUserStore()

const requests = requestService()
const routing = routingService()

const userIsFavorite = ref<ExistsInFavoritesResponse>()

const userConteiner = ref<HTMLDivElement | null>(null)

const isCurrentUserItem = ref<boolean>(false)

const props = defineProps<{
  user: CurrentUser | User,
  backgroundColor?: string
}>()

const emit = defineEmits<{
  (e: 'press'): void,
  (e: 'followed'): void,
  (e: 'unfollowed'): void
}>()

onMounted(async () => {
  userIsFavorite.value = await requests.checkExistsInFavorites(props.user.id)
  const currentUser: CurrentUser | null = getCurrentUserData()
  isCurrentUserItem.value = currentUser?.id === props.user.id
  if (props.backgroundColor && userConteiner.value) {
    userConteiner.value.style.setProperty('background-color', props.backgroundColor)
  }
})

async function followUser() {
  if (props.user) {
    await requests.addToFavorites(props.user.id)
    userIsFavorite.value = await requests.checkExistsInFavorites(props.user.id)
    emit('followed')
  }
}

async function unfollowUser() {
  if (props.user) {
    await requests.removeFromFavorites(props.user.id)
    userIsFavorite.value = await requests.checkExistsInFavorites(props.user.id)
    emit('unfollowed')
  }
}

function routToUserProfile() {
  if (props.user) {
    emit('press')
    if (isCurrentUserItem.value) {
      routing.toProfile()
    } else {
      routing.toUserProfile(props.user.id)
    }
  }
}
</script>

<style scoped lang="scss">
.creator {

  &__container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px;
    border-radius: 8px;
    background-color: #ebe7e7;
  }

  &__avatar {
    width: 50px;
    height: 50px;
    margin-right: 10px;

    img {
      border: 3px solid var(--app-red);
      border-radius: 50%;
      background-color: #fff;
    }
  }

  &__name {
    width: 110px;
    font-size: 20px;
    font-weight: 500;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}

.data__wrapper {
  display: flex;
  align-items: center 
}
</style>
