<template>
  <div class="hidden text-red-800 text-red-700 border-red-500 bg-red-50"></div>
  <div class="hidden text-yellow-800 text-yellow-700 border-yellow-500 bg-yellow-50"></div>
  <div class="hidden text-green-800 text-green-700 border-green-500 bg-green-50"></div>
  <div class="mt-16 md:flex md:flex-wrap justify-center gap-y-4 gap-x-6" id="app">
    <div class="block md:inline-flex rounded-lg border border-slate-100 bg-slate-50 p-2">
      <div class="bg-white">
        <label
          for="domain"
          class="relative block overflow-hidden border-b border-gray-200 pt-3 focus-within:border-blue-600"
        >
          <input
            type="text"
            name="domain"
            v-model="state.domain"
            class="peer h-8 w-full border-none bg-transparent p-2 placeholder-transparent focus:border-transparent focus:outline-none focus:ring-0 sm:text-sm"
          />
        
          <span
            class="absolute left-0 top-2 -translate-y-1/2 text-xs text-gray-700 transition-all peer-placeholder-shown:top-1/2 peer-placeholder-shown:text-sm peer-focus:top-2 peer-focus:text-xs p-2"
          >
            Your Domain
          </span>
        </label>
      </div>
      <div
        class="inline-block rounded border border-slate-100 bg-transparent px-12 py-3 font-medium text-slate-300 text-xsm"
      >
        Forwards To
      </div>
      <div class="bg-white">
        <label
          for="email"
          class="relative block overflow-hidden border-b border-gray-200 pt-3 focus-within:border-blue-600"
        >
          <input
            type="text"
            name="email"
            v-model="state.email"
            class="peer h-8 w-full border-none bg-transparent p-2 placeholder-transparent focus:border-transparent focus:outline-none focus:ring-0 sm:text-sm"
          />
        
          <span
            class="absolute left-0 top-2 -translate-y-1/2 text-xs text-gray-700 transition-all peer-placeholder-shown:top-1/2 peer-placeholder-shown:text-sm peer-focus:top-2 peer-focus:text-xs p-2"
          >
            Your Email
          </span>
        </label>
      </div>
      <button class="inline-block rounded bg-gradient-to-r from-pink-500 via-red-500 to-yellow-500 p-[2px] hover:text-white focus:outline-none focus:ring active:text-opacity-75 mx-4 mt-4 md:mt-0" @click="submit($event)">
        <span class="block rounded-sm bg-white px-8 py-3 text-sm font-medium hover:bg-transparent" >
          Get a free alias
        </span>
    </button>
    </div>
    <div :role="state.msg_level" class="rounded border-l-4 p-4" :class="[!state.first_time && (fields_missing || state.show_msg) ? '' : 'hidden', state.msg_box_colors]">
      <strong class="block font-medium" :class="state.msg_header_text_color"> {{ state.msg_header }}</strong>
      <p class="mt-2 text-sm" :class="state.msg_body_text_color">
         {{ state.msg_body }}
      </p>
    </div>
  </div>
</template>
<script setup>
  import { reactive, computed, watch } from "vue";
  const state = reactive({
    email: undefined,
    domain: undefined,
    first_time: true,
    email_placeholder: 'e.g. rishi.vardhan@gmail.com',
    domain_placeholder: 'e.g. incredible-me.org',
    user: undefined,
    msg_level: 'alert',
    msg_header: 'Fields Missing',
    msg_body: 'Your domain, email, or both are missing.',
    msg_header_text_color: 'text-red-800',
    msg_body_text_color: 'text-red-700',
    msg_box_colors: 'border-red-500 bg-red-50',
    show_msg: false,
  });
  const fields_missing = computed(() => {
    return state.email && state.domain? false : true;
  }) 
  watch(
    () => state.user,
    (n, o) => {
      console.log(`watched ${JSON.stringify(n)}`);
      if(n && n.type){
        let baseColor = 'red';
        switch(n.type){
            case 'existing_user_domain': 
              state.msg_level = 'info';
              state.msg_header = 'Login';
              state.msg_body = 'This domain already forwards to your email. Login instead.';
              state.show_msg = true;
              baseColor = 'green';
              break;
            case 'existing_user:free': 
              state.msg_level = 'warning';
              state.msg_header = 'Upgrade';
              state.msg_body = 'Your email is already receiving forwards from a different domain. Upgrade instead?';
              state.show_msg = true;
              baseColor = 'yellow';
              break;
            case 'existing_domain': 
              state.msg_level = 'warning';
              state.msg_header = 'Domain in use';
              state.msg_body = 'This domain is already forwarding emails elsewhere';
              state.show_msg = true;
              baseColor = 'yellow';
              break;
            case 'no_user_domain': 
              state.msg_level = 'info';
              state.msg_header = 'Welcome';
              state.msg_body = 'Your forwarding has been created. Check your inbox for more details.';
              state.show_msg = true;
              baseColor = 'green';
              break;
            case 'domain_not_registered': 
              state.msg_level = 'alert';
              state.msg_header = 'Domain not registered';
              state.msg_body = 'This domain does not seem to be registered.';
              state.show_msg = true;
              baseColor = 'red';
              break;
        }
        state.msg_header_text_color = `text-${baseColor}-800`;
        state.msg_body_text_color = `text-${baseColor}-700`;
        state.msg_box_colors = `border-${baseColor}-500 bg-${baseColor}-50`;
      }
    },
    { deep: true }
  )
  async function submit(event) {
    console.log(`fields missing val = ${fields_missing.value}`);
    state.first_time = false;
    state.show_msg = false;
    if(!fields_missing.value){
      await fetch("https://api.pretzelbox.cc/p/register-my-domain?" + new URLSearchParams({
            email: state.email,
            domain: state.domain,
        }), {
        method: "GET",
        headers: {"x-api-key": "GKAJH032au6jLEMlSUZB35iJCrGQs3sq2dH2cyuQ"},
      })
      .then(r => r.json())
      .then(r => {
        console.log(r);
        state.user = r;
      })
      .catch(e => {
        console.log(e);
      });
    }
  }
</script>