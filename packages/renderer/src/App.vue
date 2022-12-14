<template>
  <app-command-prompt />
  <app-sidebar v-if="!store.inFocusMode" />
  <main v-if="retrieved" :class="{ 'pl-16': !store.inFocusMode }">
    <router-view />
  </main>
  <ui-dialog />
</template>
<script>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useTheme } from './composable/theme';
import { useStore } from './store';
import { useNoteStore } from './store/note';
import { useLabelStore } from './store/label';
import notes from './utils/notes';
import AppSidebar from './components/app/AppSidebar.vue';
import AppCommandPrompt from './components/app/AppCommandPrompt.vue';

export default {
  components: { AppSidebar, AppCommandPrompt },
  setup() {
    const theme = useTheme();
    const store = useStore();
    const router = useRouter();
    const noteStore = useNoteStore();
    const labelStore = useLabelStore();

    const retrieved = ref(false);

    const isFirstTime = localStorage.getItem('first-time');

    if (!isFirstTime) {
      const promises = Promise.allSettled(
        Object.values(notes).map(({ title, content, id }) =>
          noteStore.add({ id, title, content: JSON.parse(content) })
        )
      );

      labelStore.add('Introduction');
      labelStore.add('Tutorial');

      promises.then(() => {
        const note = noteStore.notes.find(
          ({ title }) => title === 'Welcome to Notething'
        );

        if (note) router.push(`/note/${note.id}`);

        localStorage.setItem('first-time', false);
        retrieved.value = true;
      });
    } else {
      store.retrieve().then(() => (retrieved.value = true));

      const lastNoteEdit = localStorage.getItem('lastNoteEdit');

      if (lastNoteEdit) {
        router.push(`/note/${lastNoteEdit}`);
      }
    }

    theme.loadTheme();

    return {
      store,
      retrieved,
    };
  },
};
</script>
