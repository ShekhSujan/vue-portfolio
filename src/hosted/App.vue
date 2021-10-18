<template>
  <main>
    <h3 style="text-align:center"><a href="#"><img
      src="../images/profile.jpg"

      style="width: 70px;border-radius: 50px;"/></a></h3>

    <vue-command
      :autocompletion-resolver="autocompletionResolver"
      :built-in="builtIn"
      :commands="commands"
      :cursor.sync="cursor"
      :history.sync="history"
      :help-timeout="1250"
      :prompt="prompt"
      :stdin.sync="stdin"
      show-help>
    </vue-command>
    <p> *** Type <b>help</b> and browse through available commands!</p>
  </main>
</template>

<script>

import VueCommand from '../components/VueCommand'
import { createStdout, createStderr, createDummyStdout } from '../library'

const PROMPT = '~mr@sujan:#/'

export default {
  components: {
    VueCommand
  },

  data: () => ({
    autocompletionResolver: () => undefined,
    builtIn: undefined,
    commands: {
      // Navigate to home, self and back
      cd: undefined,

      // Clear terminals history
      clear: undefined,

      // Returns the parsed object to test parsing
      // E. g.: echo --x="one two three" --y="one two" --z="one" --test="okay" --x1 --y2 --t=ok -dash
      echo: _ => createStdout(JSON.stringify(_, null, 2)),

      // Show help
      help: () => createStdout(`Available Commands:<br>
       
        &nbsp;<span class='ab'><b>about</b></span><br>
        &nbsp;<span class='ab'><b>contact</b></span><br>
        &nbsp;<span class='ab'><b>link</b></span><br>
        &nbsp;<span class='ab'><b>clear</b></span><br>
   
      `),

      // Return simple text
      about: () => createStdout(`
        &nbsp;<span class='ef'>Always focus on what’s important, Dedicated & Passonate About Work! ♓</span><br>
        &nbsp;<b class='cd'>Exp:</b> <span class='ef'>2 Yrs +</span><br>
        &nbsp;<b class='cd'>Skills:</b> <span class='ef'>PHP, Laravel, MysQl, Jquery, VueJs..</span><br>
        &nbsp;<b class='cd'>Hobby:</b> <span class='ef'>Coding, Reading Books, Watching Movies, Travelling..</span><br>
      `),
      contact: () => createStdout(`
       
        &nbsp;<b class='cd'>Mobile:</b> <span class='ef' target='_blank'>01792295656,01910005054</span><br>
        &nbsp;<b class='cd'>Email:</b> <span class='ef' target='_blank'>sujanmahmudovi@gmail.com</span><br>
        &nbsp;<b class='cd'>Address:</b> <span class='ef' target='_blank'>Dhaka, Bangladesh</span><br>
      `),
      link: () => createStdout(`
       
        &nbsp;<b><a class='ab' target='_blank' href='https://github.com/SujanMahmudOvi'>Github</a>,
         <a class='ab' target='_blank' href='#'>StackOverFlow</a>
         </b><br>
        &nbsp;<b class='cd'>SocialMedia:</b>
         <ul>
         <li><a class='ef' target='_blank' href="https://www.facebook.com/sjn97825">FaceBook</a></li>
         <li><a class='ef' target='_blank' href="https://www.linkedin.com/in/sjn97825/">LinkedIn</a></li>
         </ul>
        &nbsp;<b class='cd'>Projects:</b>
         <ul>
         <li><a class='ef' target='_blank' href="#">Event ManageMent System</a></li>
         <li><a class='ef' target='_blank' href="#">Hospital Management System </a></li>
         </ul>
         <br>
      `)

    },

    // Terminal cursor position
    cursor: 0,
    history: [],
    options: {
      long: {
        pokedex: ['color'],
        loading: ['amount', 'timeout']
      },

      short: {
        pokedex: ['h']
      }
    },

    prompt: PROMPT,
    stdin: ''
  }),

  created () {
    this.commands.clear = () => {
      this.history = []
      // Push dummy Stdout to show Stdin
      return createDummyStdout()
    }

    this.commands.cd = ({ _ }) => {
      if ((_[1] === 'home' || _[1] === 'home/') && this.prompt === PROMPT) {
        this.prompt = `${PROMPT}home`

        return createDummyStdout()
      }

      // Navigate from home to root
      if ((_[1] === '../' || _[1] === '..') && this.prompt === `${PROMPT}home`) {
        this.prompt = PROMPT

        return createDummyStdout()
      }

      // Navigate to self
      if (_[1] === '.' || _[1] === './' || typeof _[1] === 'undefined') {
        return createDummyStdout()
      }

      return createStderr(`cd: ${_[1]}: No such file or directory or have not allowed! 😢`)
    }

    // this.commands.pwd = () => {
    //   // Take current prompt into account
    //   if (this.prompt === '~mr@sujan:#/') {
    //     return createStdout('This is root path 👍')
    //   } else {
    //     return createStdout('/home')
    //   }
    // }

    this.builtIn = (stdin, terminal) => {
      // Check for application
      if (stdin.trim().split(' ')[0] !== 'reverse') {
        terminal.commandNotFound(stdin)

        return
      }

      stdin = stdin.trim()
      // Get second argument
      const argument = stdin.split(' ').slice(1).join(' ').replace(/"/g, '')

      // Do nothing if no argument given
      if (!argument) {
        return
      }

      // Reverse argument
      this.stdin = argument.split('').reverse().join('')
    }

    this.autocompletionResolver = () => {
      // Preserve cursor position
      const cursor = this.cursor

      // Reverse concatenate autocompletable according to cursor
      let pointer = this.cursor
      let autocompleteableStdin = ''
      while (this.stdin[pointer - 1] !== ' ' && pointer - 1 > 0) {
        pointer--
        autocompleteableStdin = `${this.stdin[pointer]}${autocompleteableStdin}`
      }

      // Divide by arguments
      const command = this.stdin.split(' ')

      // Autocompleteable is program
      if (command.length === 1) {
        const autocompleteableProgram = command[0]
        // Collect all autocompletion candidates
        const candidates = []
        const programs = [...Object.keys(this.commands), 'reverse'].sort()
        programs.forEach(program => {
          if (program.startsWith(autocompleteableProgram)) {
            candidates.push(program)
          }
        })

        // Autocompletion resolved into multiple results
        if (this.stdin !== '' && candidates.length > 1) {
          this.history.push({
            // Build table programmatically
            render: createElement => {
              const columns = candidates.length < 5 ? candidates.length : 4
              const rows = candidates.length < 5 ? 1 : Math.ceil(candidates.length / columns)

              let index = 0
              const table = []
              for (let i = 0; i < rows; i++) {
                const row = []
                for (let j = 0; j < columns; j++) {
                  row.push(createElement('td', candidates[index]))
                  index++
                }

                table.push(createElement('tr', [row]))
              }

              return createElement('table', { style: { width: '100%' } }, [table])
            }
          })
        }

        // Autocompletion resolved into one result
        if (candidates.length === 1) {
          // Mutating Stdin mutates the cursor, so we've to wait to push it to the end
          const unwatch = this.$watch(() => this.cursor, () => {
            this.cursor = cursor + (candidates[0].length - autocompleteableStdin.length + 0)

            unwatch()
          })

          this.stdin = candidates[0]
        }

        return
      }

      // Check if option might be completed already or option is last tokens
      if ((this.stdin[cursor] !== '' && this.stdin[cursor] !== ' ') && typeof this.stdin[cursor] !== 'undefined') {
        return
      }

      // Get the executable
      const program = command[0]

      // Check if any autocompleteable exists
      if (typeof this.options.long[program] === 'undefined' && typeof this.options.short[program] === 'undefined') {
        return
      }

      // Autocompleteable is long option
      if (autocompleteableStdin.substring(0, 2) === '--') {
        const candidates = []
        this.options.long[program].forEach(option => {
          // If only dashes are presents, user requests all options
          if (`--${option}`.startsWith(autocompleteableStdin) || autocompleteableStdin === '--') {
            candidates.push(option)
          }
        })

        // Autocompletion resolved into one result
        if (candidates.length === 1) {
          const autocompleted = `${this.stdin.substring(0, pointer - 1)} --${candidates[0]}`
          const rest = `${this.stdin.substring(this.cursor)}`

          // Mutating Stdin mutates the cursor, so we've to wait to push it to the end
          const unwatch = this.$watch(() => this.cursor, () => {
            this.cursor = cursor + (candidates[0].length - autocompleteableStdin.length + 2)

            unwatch()
          })

          this.stdin = `${autocompleted}${rest}`

          return
        }

        // Autocompletion resolved into multiple result
        if (autocompleteableStdin === '--' || candidates.length > 1) {
          this.history.push({
            // Build table programmatically
            render: createElement => {
              const columns = candidates.length < 5 ? candidates.length : 4
              const rows = candidates.length < 5 ? 1 : Math.ceil(candidates.length / columns)

              let index = 0
              const table = []
              for (let i = 0; i < rows; i++) {
                const row = []
                for (let j = 0; j < columns; j++) {
                  row.push(createElement('td', `--${candidates[index]}`))
                  index++
                }

                table.push(createElement('tr', [row]))
              }

              return createElement('table', { style: { width: '100%' } }, [table])
            }
          })
        }

        return
      }

      // Autocompleteable is option
      if (autocompleteableStdin.substring(0, 1) === '-') {
        const candidates = []
        this.options.short[program].forEach(option => {
          // If only one dash is present, user requests all options
          if (`-${option}`.startsWith(autocompleteableStdin) || autocompleteableStdin === '-') {
            candidates.push(option)
          }
        })

        // Autocompletion resolved into one result
        if (candidates.length === 1) {
          const autocompleted = `${this.stdin.substring(0, pointer - 1)} -${candidates[0]}`
          const rest = `${this.stdin.substring(this.cursor)}`

          // Mutating Stdin mutates the cursor, so we've to wait to push it to the end
          const unwatch = this.$watch(() => this.cursor, () => {
            this.cursor = cursor + (candidates[0].length - autocompleteableStdin.length + 1)

            unwatch()
          })

          this.stdin = `${autocompleted}${rest}`

          return
        }

        // Autocompletion resolved into multiple result
        if (autocompleteableStdin === '-' || candidates.length > 1) {
          this.history.push({
            // Build table programmatically
            render: createElement => {
              const columns = candidates.length < 5 ? candidates.length : 4
              const rows = candidates.length < 5 ? 1 : Math.ceil(candidates.length / columns)

              let index = 0
              const table = []
              for (let i = 0; i < rows; i++) {
                const row = []
                for (let j = 0; j < columns; j++) {
                  row.push(createElement('td', `-${candidates[index]}`))
                  index++
                }

                table.push(createElement('tr', [row]))
              }

              return createElement('table', { style: { width: '100%' } }, [table])
            }
          })
        }
      }
    }
  }
}
</script>

<style lang="scss">
$border-radius: 8px;

body {
  display: grid;
  place-items: center;
  height: 80vh;
  margin: 0;
  background: #bbccd8;
// background-image: url("../images/b.webp");
  main {
    margin: 1rem;
    max-width: 550px;
    width: calc(100% - 2rem);
  }

  h1,
  h2,
  h3 {
    font-family: "Inconsolata", monospace;
  }

  p {
    font-family: "Montserrat", sans-serif;
  }

  pre {
    width: 100%;
    padding: 0;
    margin-top: 1em;
    overflow: auto;
    overflow-y: hidden;

    code {
      padding: 10px;
      color: #333;
      margin: 5px;
    }
  }

  .vue-command {
    -webkit-border-bottom-left-radius: $border-radius;
    -webkit-border-bottom-right-radius: $border-radius;
    -moz-border-bottom-left-radius: $border-radius;
    -moz-border-bottom-right-radius: $border-radius;
    border-bottom-left-radius: $border-radius;
    border-bottom-right-radius: $border-radius;

    ::-webkit-scrollbar {
      width: 6px;
    }

    ::-webkit-scrollbar-track {
      background: #252525;
    }

    ::-webkit-scrollbar-thumb {
      background: #ddbdbd;
    }

    ::-webkit-scrollbar-thumb:hover {
      background: #333;
    }

    .term-bar {
      -webkit-border-top-left-radius: $border-radius;
      -webkit-border-top-right-radius: $border-radius;
      -moz-border-top-right-radius: $border-radius;
      -moz-border-top-left-radius: $border-radius;
      border-top-left-radius: $border-radius;
      border-top-right-radius: $border-radius;
    }

    .term-std {
      min-height: 312px;
      max-height: 312px;
      overflow-y: scroll;
    }
    .term-stdout{
      font-size: 12px;
    }
    .ab{
      text-decoration: none;
      color: #d980b5;
    }
    .cd{
      color: #ef8e93;
    }
    .ef{
        text-decoration: none;
      color: #a7bd83;
    }
    .profile{
      border-radius: 55px;
    }
  }
}
</style>
