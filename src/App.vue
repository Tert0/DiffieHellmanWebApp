<template>
  <div id="app">
    <table>
      <tr>
        <th>Alice</th>
        <th>Public Numbers</th>
        <th>Bob</th>
      </tr>

      <tr>
        <td>
          Alice Private Key: {{ alice_private }}
          <br>
          <button @click="alice_private = generatePrivateKey(public_g)">Generate</button>
        </td>
        <td>
          G={{ public_g }}
        </td>
        <td>
          Bob Private Key: {{ bob_private }}
          <br>
          <button @click="bob_private = generatePrivateKey(public_g)">Generate</button>
        </td>
      </tr>

      <tr>
        <td></td>
        <td>
          p={{ public_p }}
          <br>
          <button @click="public_p = generateP();public_g = generateG()">Generate</button>
        </td>
        <td></td>
      </tr>

      <tr>
        <td>Alice Public Key: {{ public_g }}<sup>{{ alice_private }}</sup> mod {{ public_p }} = {{ alice_public }}</td>
        <td></td>
        <td>Bob Public Key: {{ public_g }}<sup>{{ bob_private }}</sup> mod {{ public_p }} = {{ bob_public }}</td>
      </tr>

      <tr>
        <td>Shared Secret: {{ bob_public }}<sup>{{ alice_private }}</sup> mod {{ public_p }} = {{ alice_shared_secret }}
        </td>
        <td></td>
        <td>Shared Secret: {{ alice_public }}<sup>{{ bob_private }}</sup> mod {{ public_p }} = {{ bob_shared_secret }}
        </td>
      </tr>
    </table>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
  name: 'App',
  data() {
    return {
      alice_private: 0,
      bob_private: 0,
      public_g: 5,
      public_p: 23,
    }
  },
  methods: {
    generatePrivateKey(g: number): number {
      let key = Math.floor(Math.random() * g);
      return key != 0 ? key : 1;
    },
    generateG(): number {
      return this.findPrimitive(this.public_p);
    },
    getPrimes(min: number, max: number): Array<number> {
      const result = Array(max + 1)
          .fill(0)
          .map((_, i) => i);
      for (let i = 2; i <= Math.sqrt(max + 1); i++) {
        for (let j = i ** 2; j < max + 1; j += i) delete result[j];
      }
      return Object.values(result.slice(min));
    },
    generateP(): number {
      const primes: Array<number> = this.getPrimes(100, 200);
      return primes[Math.floor(Math.random() * primes.length + 1) - 1];
    },
    isPrime(n: number): boolean {
      if (n <= 1)
        return false;
      if (n <= 3)
        return true;

      if (n % 2 == 0 || n % 3 == 0)
        return false;

      for (let i = 5; i * i <= n; i = i + 6)
        if (n % i == 0 || n % (i + 2) == 0)
          return false;

      return true;
    },
    power(x: number, y: number, p: number): number {
      let res = 1;

      x = x % p;

      while (y > 0) {
        if (y & 1)
          res = (res * x) % p;

        y = y >> 1;
        x = (x * x) % p;
      }
      return res;
    },
    findPrimefactors(s: Set<unknown>, n: number): void {
      while (n % 2 == 0) {
        s.add(2);
        n = n / 2;
      }

      for (let i = 3; i <= Math.sqrt(n); i = i + 2) {
        while (n % i == 0) {
          s.add(i);
          n = n / i;
        }
      }

      if (n > 2)
        s.add(n);
    },
    findPrimitive(n: number): number {
      let s = new Set();
      if (!this.isPrime(n))
        return -1;
      let phi = n - 1;

      this.findPrimefactors(s, phi);

      for (let r = Math.floor(phi/2); r > 2; r--) {
        let flag = false;
        for (let it of s) {
          console.log(`R: ${r} Y: ${phi / Number(it)} N: ${n} Result: ${this.power(r, phi / Number(it), n)}`)
          if (this.power(r, phi / Number(it), n) == 1) {
            flag = true;
            break;
          }
        }

        if (!flag)
          return r;
      }

      return -1;
    }
  },
  mounted() {
    this.public_p = this.generateP()
    this.public_g = this.generateG();
    this.alice_private = this.generatePrivateKey(this.public_g);
    this.bob_private = this.generatePrivateKey(this.public_g);
  },
  computed: {
    alice_public: function (): number {
      return Math.pow(this.public_g, this.alice_private) % this.public_p;
    },
    bob_public: function (): number {
      return Math.pow(this.public_g, this.bob_private) % this.public_p;
    },
    alice_shared_secret: function (): number {
      return Math.pow(this.bob_public, this.alice_private) % this.public_p;
    },
    bob_shared_secret: function (): number {
      return Math.pow(this.alice_public, this.bob_private) % this.public_p;
    }
  }
});
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
}

table {
  margin-left: 12.5%;
  width: 75%;
  border-collapse: collapse;

  td, th {
    border: 1px solid #eeeeee;
    text-align: left;
    padding: 8px;
  }
}

.same_secrets {
  border: 1px solid #1f950d;
}
</style>
