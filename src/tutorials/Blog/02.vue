<template>
  <Lesson
    :text="text"
    :code="code"
    :validate="validate"
    :exercise="exercise"
    :solution="solution"
    lessonTitle="Update posts with tags and watch their CIDs change" />
</template>

<script>
import Lesson from '../../components/Lesson'
import text from './02.md'
import exercise from './02-exercise.md'
import utils from './utils.js'
import shallowEqualArrays from 'shallow-equal/arrays'

const code = `/* globals ipfs */

const run = async () => {
  const natCid = await ipfs.dag.put({ author: "Nat" })
  const samCid = await ipfs.dag.put({ author: "Sam" })

  // Modify the blog posts below
  const treePostCid = await ipfs.dag.put({
    content: "trees",
    author: samCid
  })
  const computerPostCid = await ipfs.dag.put({
    content: "computers",
    author: natCid
  })

  // DELETE ME

  return [treePostCid, computerPostCid]
}

return run`

const validate = async (result, ipfs) => {
  if (!result) {
    return { fail: 'You forgot to return a result :)' }
  }

  const validatedArray = utils.validateArrayOfCids(result, 2)
  if (validatedArray.fail) {
    return validatedArray
  }

  const TREE_POST_CID = 'zdpuAkSPEnmgR1rqKkzpFN5qfJshCQKqMaVtUSpQJAMLdw3KF'
  const COMPUTER_POST_CID = 'zdpuAxzw762rP3CXZpAsKagPFR2AyqmZU2sN8U1GuVCeoYUEo'
  if (TREE_POST_CID === result[0].toBaseEncodedString() && COMPUTER_POST_CID === result[1].toBaseEncodedString()) {
    return {
      log: {
        treePostCid: result[0].toBaseEncodedString(),
        computerPostCid: result[1].toBaseEncodedString()
      }
    }
  }

  for (const cid of result) {
    const obj = await ipfs.dag.get(cid)
    const node = obj.value
    if (node.tags === undefined) {
      return { fail: 'Blog posts need to have a `tags` field.' }
    }
    if (!Array.isArray(node.tags)) {
      return { fail: 'The value of the `tags` field must be an array of strings.' }
    }
    const isStrings = node.tags.every((tag) => typeof tag === 'string')
    if (!isStrings) {
      return { fail: `Tags need to be strings.` }
    }
    let expectedTags
    switch (node.content) {
      case 'trees':
        expectedTags = ['hobby', 'outdoor']
        break
      case 'computers':
        expectedTags = ['hobby']
        break
    }
    if (!shallowEqualArrays(node.tags.sort(), expectedTags.sort())) {
      return { fail: `The tags of the "${node.content}" blog post ${utils.stringify(node.tags)} did not match the the expected tags ${utils.stringify(expectedTags)}.` }
    }
  }

  // Don't check the CIDs as then the order of the tags would matter.
  // But that order really doesn't matter.
  return {
    success: 'Everything works!',
    logDesc: 'These are the CIDs of the blog posts. Notice how they change when the underlying data is altered.',
    log: {
      treePostCid: result[0].toBaseEncodedString(),
      computerPostCid: result[1].toBaseEncodedString()
    }
  }
}

const solution = `/* globals ipfs */

const run = async () => {
  const natCid = await ipfs.dag.put({ author: "Nat" })
  const samCid = await ipfs.dag.put({ author: "Sam" })

  const treePostCid = await ipfs.dag.put({
    content: "trees",
    author: samCid,
    tags: ["outdoor", "hobby"]
  })
  const computerPostCid = await ipfs.dag.put({
    content: "computers",
    author: natCid,
    tags: ["hobby"]
  })

  return [treePostCid, computerPostCid]
}

return run
`

export default {
  components: {
    Lesson
  },
  data: () => {
    return { code, text, validate, exercise, solution }
  }
}
</script>
