<script setup>
import logo from '../assets/logo.png'
import starburst from '../assets/starburst.png'

import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { signInWithEmailAndPassword } from 'firebase/auth'
import { hostAuth, hostFirestore } from '../hostfirebase'
import { doc, getDoc } from 'firebase/firestore'
import { useAuthStore } from '../stores/auth' 

const authStore = useAuthStore()
const email = ref('')
const password = ref('')
const message = ref('')
const router = useRouter()

const loginHost = async () => {
  message.value = ''
  try {
    const result = await signInWithEmailAndPassword(hostAuth, email.value, password.value)
    const user = result.user

    const docRef = doc(hostFirestore, 'hosts', user.uid)
    const docSnap = await getDoc(docRef)
    const userData = docSnap.exists() ? docSnap.data() : {}

    authStore.setUser({
      email: user.email,
      uid: user.uid,
      displayName: user.displayName,
      photoURL: user.photoURL,
      role: userData.role || 'default'
    })

    if (docSnap.exists() && docSnap.data().role === 'host') {
      message.value = `Welcome, ${user.email}`
      router.push('/host-dashboard') 
    } else {
      message.value = 'Access denied. This account is not a host.'
    }

  } catch (error) {
    message.value = error.message
  }
}

</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-[#fffff] px-4">

    <router-link
      to="/"
      class="absolute top-4 left-4 text-gray-700 bg-white px-4 py-2 rounded shadow hover:bg-gray-100 transition"
    >
      ← Back
    </router-link>

    <div class="max-w-6xl w-full grid grid-cols-1 md:grid-cols-2 gap-0 rounded-lg overflow-hidden">
      
      <div class="bg-black flex items-center justify-center p-4 rounded-lg overflow-hidden">
        <img :src="starburst" alt="Starburst" class="w-full h-auto object-contain" />
      </div>


      <div class="flex items-center justify-center p-10 bg-[#ffff]">
        <div class="w-full max-w-2xl bg-white rounded-lg shadow-[0_0_20px_rgba(0,0,0,0.5)] p-12">


          <div class="flex items-center gap-2 mb-6">
            <img :src="logo" alt="Logo" class="h-6" />
          </div>

          <h2 class="text-2xl font-semibold mb-1">Welcome Back <span class="inline-block">👋</span></h2>
          <p class="text-gray-500 text-sm mb-6">
            Today is a new day. It's your day. You shape it. Sign in to start managing your events.
          </p>

          <form @submit.prevent="loginHost" class="space-y-4">
            <div>
              <label for="email" class="block text-sm text-gray-600 mb-1">Email</label>
              <input
                id="email"
                v-model="email"
                type="email"
                placeholder="Example@email.com"
                class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-blue-400"
              />
            </div>

            <div>
              <label for="password" class="block text-sm text-gray-600 mb-1">Password</label>
              <input
                id="password"
                v-model="password"
                type="password"
                placeholder="At least 8 characters"
                class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-blue-400"
              />
              <div class="text-right mt-1">
                <a href="#" class="text-xs text-blue-400 hover:underline">Forgot Password?</a>
              </div>
            </div>

            <button type="submit" class="w-full bg-[#0f1f2e] hover:bg-[#1a2b3d] text-white py-2 rounded shadow">
              Sign in
            </button>
          </form>

          <p class="text-sm text-red-500 mt-3 text-center">{{ message }}</p>


          <p class="text-sm text-center text-gray-500 mt-6">
            Don’t you have an account?
            <router-link to="/host-signup" class="text-blue-500 hover:underline">Sign up</router-link>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>
