<script setup lang="ts">
import { RouterView } from 'vue-router'
import { Suspense } from 'vue'

</script>

<template> 
    <main class="bg-[#F3EBF6]">
        <Suspense>
            <RouterView />
        </Suspense>
    </main>
</template>
