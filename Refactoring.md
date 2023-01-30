# Refactoring

You've been asked to refactor the function `deterministicPartitionKey` in [`dpk.js`](dpk.js) to make it easier to read and understand without changing its functionality. For this task, you should:

1. Write unit tests to cover the existing functionality and ensure that your refactor doesn't break it. We typically use `jest`, but if you have another library you prefer, feel free to use it.
2. Refactor the function to be as "clean" and "readable" as possible. There are many valid ways to define those words - use your own personal definitions, but be prepared to defend them. Note that we do like to use the latest JS language features when applicable.
3. Write up a brief (~1 paragraph) explanation of why you made the choices you did and why specifically your version is more "readable" than the original.

You will be graded on the exhaustiveness and quality of your unit tests, the depth of your refactor, and the level of insight into your thought process provided by the written explanation.

## Your Explanation Here
const crypto = require("crypto");

const TRIVIAL_PARTITION_KEY = "0";
const MAX_PARTITION_KEY_LENGTH = 256;

exports.deterministicPartitionKey = (event = {}) => {
  let candidate = event.partitionKey || 
    crypto.createHash("sha3-512").update(JSON.stringify(event)).digest("hex");
  
  if (typeof candidate !== "string") {
    candidate = JSON.stringify(candidate);
  }
  return candidate.length > MAX_PARTITION_KEY_LENGTH ?
    crypto.createHash("sha3-512").update(candidate).digest("hex") :
    candidate || TRIVIAL_PARTITION_KEY;
};


I refactored the code to reduce the number of if-else statements and make the logic more concise. The code now takes an optional event argument with a default value of an empty object, this reduces the amount of null checks. The candidate value is now set using a ternary operator, which makes the code more concise and readable. The candidate value is also now only converted to a string if it is not already a string. Finally, I reduced the amount of code duplication by using the candidate value in both branches of the final ternary operator.

Unit tests:

describe("deterministicPartitionKey", () => {
  it("should return trivial partition key if event is not provided", () => {
    expect(deterministicPartitionKey()).toBe(TRIVIAL_PARTITION_KEY);
  });
  
  it("should return partition key if event has partition key", () => {
    const partitionKey = "partition-key";
    expect(deterministicPartitionKey({ partitionKey })).toBe(partitionKey);
  });
  
  it("should return hash if event has no partition key", () => {
    const event = {};
    const hash = crypto
      .createHash("sha3-512")
      .update(JSON.stringify(event))
      .digest("hex");
    expect(deterministicPartitionKey(event)).toBe(hash);
  });
  
  it("should return hash if partition key is too long", () => {
    const partitionKey = "a".repeat(MAX_PARTITION_KEY_LENGTH + 1);
    const hash = crypto
      .createHash("sha3-512")
      .update(partitionKey)
      .digest("hex");
    expect(deterministicPartitionKey({ partitionKey })).toBe(hash);
  });
});
