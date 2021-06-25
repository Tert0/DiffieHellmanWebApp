<template>
  <div id="app">
    <table :class="{ same_secret: alice_shared_secret === bob_shared_secret, error: alice_shared_secret !== bob_shared_secret }">
      <tr>
        <th>Alice</th>
        <th>Public Numbers</th>
        <th>Bob</th>
      </tr>

      <tr>
        <td>
          Alice Private Key: {{ alice_private }}
          <br />
          <button @click="alice_private = generatePrivateKey(public_p)">
            Generate
          </button>
        </td>
        <td>G = {{ public_g }}</td>
        <td>
          Bob Private Key: {{ bob_private }}
          <br />
          <button @click="bob_private = generatePrivateKey(public_p)">
            Generate
          </button>
        </td>
      </tr>

      <tr>
        <td></td>
        <td>
          p = {{ public_p }}
          <br />
          <button
              @click="
              public_p = generateP();
              public_g = generateG();
            "
          >
            Generate
          </button>
        </td>
        <td></td>
      </tr>

      <tr>
        <td>
          Alice Public Key: {{ public_g }}<sup>{{ alice_private }}</sup> mod
          {{ public_p }} = {{ alice_public }}
        </td>
        <td></td>
        <td>
          Bob Public Key: {{ public_g }}<sup>{{ bob_private }}</sup> mod
          {{ public_p }} = {{ bob_public }}
        </td>
      </tr>

      <tr>
        <td>
          Shared Secret: {{ bob_public }}<sup>{{ alice_private }}</sup> mod
          {{ public_p }} = {{ alice_shared_secret }}
        </td>
        <td></td>
        <td>
          Shared Secret: {{ alice_public }}<sup>{{ bob_private }}</sup> mod
          {{ public_p }} = {{ bob_shared_secret }}
        </td>
      </tr>
    </table>
  </div>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "App",
  data() {
    return {
      alice_private: 0,
      bob_private: 0,
      public_g: 5,
      public_p: 23
    };
  },
  methods: {
    generatePrivateKey(p: number): number {
      return Math.floor(Math.random() * (p - 1)) + 1;
    },
    generateG(): number {
      return this.findPrimitiveRootModulo(this.public_p);
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
      let primes: Array<number> = this.getPrimes(5, 40);
      primes[primes.indexOf(31)] = 11; // Work A Round
      primes[primes.indexOf(31)] = 13;
      return primes[Math.floor(Math.random() * primes.length + 1) - 1];
    },
    isPrime(n: number): boolean {
      if (n <= 1) return false;
      if (n <= 3) return true;

      if (n % 2 == 0 || n % 3 == 0) return false;

      for (let i = 5; i * i <= n; i = i + 6)
        if (n % i == 0 || n % (i + 2) == 0) return false;

      return true;
    },
    power(x: number, y: number, p: number): number {
      let res = 1;

      x = x % p;

      while (y > 0) {
        if (y & 1) res = (res * x) % p;

        y = y >> 1;
        x = (x * x) % p;
      }
      return res;
    },
    isPrimitiveRootModulo(g: number, n: number): boolean {
      let found = new Set();
      for (let i = n * 1000; i > 0  /*hack - would need to detect cycles*/; i--) {
        found.add(Math.pow(g, i) % n);
      }
      for (let i = 1; i < n; i++) {
        if (!found.has(i)) {
          return false;
        }
      }
      return true;
    },
    findPrimitiveRootModulo(n: number): number {
      for (let i = 1; i < n; i++) {
        if (this.isPrimitiveRootModulo(i, n)) {
          return i;
        }
      }
      return -1;
    }
  },
  mounted() {
    this.public_p = this.generateP();
    this.public_g = this.generateG();
    this.alice_private = this.generatePrivateKey(this.public_p);
    this.bob_private = this.generatePrivateKey(this.public_p);
  },
  computed: {
    alice_public: function(): number {
      return Math.pow(this.public_g, this.alice_private) % this.public_p;
    },
    bob_public: function(): number {
      return Math.pow(this.public_g, this.bob_private) % this.public_p;
    },
    alice_shared_secret: function(): number {
      return Math.pow(this.bob_public, this.alice_private) % this.public_p;
    },
    bob_shared_secret: function(): number {
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

  td,
  th {
    border: 1px solid #eeeeee;
    text-align: left;
    padding: 8px;
  }
}

.same_secret {
  border-collapse: collapse;
  td,
  th {
    border: 2px solid #1f950d;
  }
}

.error {
  border-collapse: collapse;
  td,
  th {
    border: 2px solid #a31010;
  }
}

button {
}
</style>
