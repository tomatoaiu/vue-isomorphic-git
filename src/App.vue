<template>
  <div id="app">
    <ul>
      <li v-for="(file, i) in files" :key="i">
        {{ file.path }}
      </li>
    </ul>
  </div>
</template>

<script>
import LightningFS from '@isomorphic-git/lightning-fs'
import { clone, log, readTree } from 'isomorphic-git'
import http from "isomorphic-git/http/web"

export default {
  name: 'App',
  data() {
    return {
      files: []
    }
  },
  async mounted() {
    const fs = new LightningFS('my-app')

    await clone({
      fs,
      http,
      dir: '/tutorial',
      corsProxy: 'https://cors.isomorphic-git.org',
      url: 'https://github.com/isomorphic-git/isomorphic-git',
      singleBranch: true,
      depth: 1,
      onProgress: event => {
        console.log("phase: ", event.phase)
        if (event.total) {
          console.log("ProgressBar: ", event.loaded / event.total)
        } else {
          console.log("IndeterminateProgressBar: ", event.loaded)
        }
      }
    })
    console.log('done')

    const commits = await log({
      fs,
      dir: '/tutorial',
      depth: 1,
      ref: 'main'
    })
    console.log(commits)

    const commitObject = commits.pop()
    const oid = commitObject.commit.tree
    const treeObject = await readTree({ fs, dir: '/tutorial', oid })
    console.log(treeObject)

    this.files = treeObject.tree.filter(o => o.type === 'blob')
  }
}
</script>
