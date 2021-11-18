<template>
  <Experiment title="Public Goods Experiment">
    <InstructionScreen :title="'Welcome, nice to see you!'">
      <p>Thank you for participating in our experiment!</p>
      <p>
        You will need around 10 min to complete the experiment. Please make sure
        that you will not be distracted. Switch off all messaging systems, your
        phone, any background music etc., and try to concentrate as much as
        possible on the task at hand.
      </p>
      <p>Click on the button below to receive instructions.</p>
    </InstructionScreen>

    <InstructionScreen :title="'General Instructions'">
      <p>
        During the course of this experiment you maintain a personal stash of
        tokens
      </p>
      <p>
        You are part of a group of real participants, who each have their own
        stash of tokens.
      </p>
      <p>
        Every round you are asked how many of your tokens you would like to
        donate to the group. The other group members do the same.
      </p>
      <p>
        Once each member has donated an amount of tokens to the group's pool,
        the donated tokens will be multiplied with a fixed factor and
        distributed evenly to all members.
      </p>
    </InstructionScreen>

    <InstructionScreen :title="'General Instructions'">
      <p>Let's play a training round.</p>
    </InstructionScreen>

    <template v-for="(reward, i) of training_rewards">
      <Screen :key="i" title="Training">
        <Slide>
          <p>Your balance is {{ training_stash }} tokens.</p>
          <p>Please select an amount to donate.</p>
          <SliderInput
            :tooltip="true"
            :max="training_stash"
            :response.sync="$magpie.measurements.amount"
            left="0"
            :right="training_stash + ''"
          />
          <template v-if="typeof $magpie.measurements.amount !== 'undefined'">
            <p>
              You are donating {{ $magpie.measurements.amount }} tokens to the
              group.
            </p>
            <button
              @click="
                training_stash =
                  training_stash - $magpie.measurements.amount + reward;
                $magpie.nextSlide();
              "
            >
              Go
            </button>
          </template>
        </Slide>

        <Slide>
          <p>Please wait for all participants to make their move.</p>
          <Wait :time="4000" @done="$magpie.nextSlide()" />
        </Slide>

        <Slide>
          <p>
            You have donated {{ $magpie.measurements.amount }} tokens and
            received {{ reward }} tokens.
          </p>
          <p>Your new balance is {{ training_stash }}</p>
          <Record :data="{ reward }" />
          <button @click="$magpie.saveAndNextScreen()">Next round</button>
        </Slide>
      </Screen>
    </template>

    <template v-for="i of 5">
      <Screen :key="i">
        <Slide>
          <WaitForParticipants
            :number="numberParticipants"
            @done="
              resetPool();
              $magpie.nextSlide();
            "
          />
          <p>Waiting for participants...</p>
        </Slide>

        <Slide>
          <p>Your balance is {{ test_stash }} tokens.</p>
          <p>Please select an amount to donate.</p>
          <SliderInput
            :tooltip="true"
            :max="test_stash"
            :response.sync="$magpie.measurements.amount"
            left="0"
            :right="test_stash + ''"
          />
          <template v-if="typeof $magpie.measurements.amount !== 'undefined'">
            <p>
              You are donating {{ $magpie.measurements.amount }} tokens to the
              group.
            </p>
            <button
              @click="
                submitDonation($magpie.measurements.amount);
                $magpie.nextSlide();
              "
            >
              Go
            </button>
          </template>
        </Slide>

        <Slide>
          <p>Please wait for all participants to make their move.</p>
          <Wait
            v-if="donated === $magpie.socket.active.length"
            :time="0"
            @done="$magpie.nextSlide()"
          />
        </Slide>

        <Slide>
          <p>
            You have donated {{ $magpie.measurements.amount }} tokens and
            received {{ reward }} tokens.
          </p>
          <p>Your new balance is {{ test_stash - amount + reward }}</p>
          <Record :data="{ reward }" />
          <button
            @click="
              $magpie.saveAndNextScreen();
              updateStash();
            "
          >
            Next round
          </button>
        </Slide>
      </Screen>
    </template>

    <PostTestScreen />

    <!-- While developing your experiment, using the DebugResults screen is fine,
      once you're going live, you can use the <SubmitResults> screen to automatically send your experimental data to the server. -->
    <DebugResultsScreen />
  </Experiment>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      training_stash: 100,
      training_rewards: [20, 23, 15, 12, 16, 8, 10, 3, 5],
      pool: 0,
      donated: 0,
      test_stash: 100,
      amount: 0,
      numberParticipants: 2 // Adjust this to fit your needs
    };
  },
  socket: {
    donate(amount) {
      this.amount = amount;
      this.pool += amount;
      this.donated++;
    }
  },
  computed: {
    reward() {
      return (this.pool * 1.3) / this.donated;
    }
  },
  methods: {
    resetPool() {
      this.pool = 0;
      this.donated = 0;
      this.amount = 0;
    },
    submitDonation(amount) {
      this.$magpie.socket.broadcast('donate', amount);
    },
    updateStash() {
      this.test_stash = this.test_stash - this.amount + this.reward;
    }
  }
};
</script>
