<script>
    import { Avatar } from '@skeletonlabs/skeleton';
    export let bubble = {
        color: 'variant-soft-primary',
        name: 'system|user',
        timestamp: '02:00 PM',
        message: 'Hello from Svelte!',
        avatar: 'https://example.com/newAvatarUrl.png',
        pid: 'pid1',
    };
</script>
<div class="grid grid-cols-[1fr_auto] gap-2 justify-end">
  <div class="card p-4 variant-soft rounded-tl-none space-y-2">
    <header class="flex justify-between items-center">
      <small class="opacity-50">{bubble.timestamp}</small>
      <p class="font-bold capitalize">{bubble.name}</p>
    </header>
    <p id="{bubble.pid}" class="font-nunito text-right">{bubble.message}</p>
  </div>
  <Avatar src="{bubble.avatar}" width="w-12" />
</div>
<br />
