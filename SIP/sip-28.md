---
sip:  27
title: Updated AT max steps and AT API
description: Updated AT max steps and AT API
author:  frank_the_tank, jjos, ohager
status: Final
type: Standard
category: Core
created: 2021-03-25
---
## Motivation
SIgnum smart contracts became much more useful with [SIP-20](sip-20.md). However, it became evident that, for more complex applications, the maximum number of steps a contract is allowed to run would need to increase. This is a simple proposal increasing the maximum number of steps a contract can run in a single block to `1'000'000`.

This is a tenfold increase compared to the current value and the impact on nodes should be minimum for any modern computer. For many cases this will even be benefical for the network, since allowing contracts to finish their running tasks in the same block reduces the amount of information that needs to be stored permanently on the blockchain. It should also be stressed that a fee is paid for every single step a contract runs.

Another limitation is the deployment of large contracts. This is not actually a protocol upgrade, but is documented here. Since becomes impossible to deploy large contracts by using only the _URI_, the option to send the contract details by the request body should be implemented.

## Specification

In the next hard-fork event, use the following code:
```java
    public long maxSteps(int height) {
        if(Burst.getFluxCapacitor().getValue(FluxValues.NEXT_FORK, height)) {
            return 1_000_000L;
        }
        return MAX_STEPS.get(atVersion(height));
    }
```

Allow to send AT creation bytes also in the body of the request, being retrieved by Signum-Node as follows:
```java
  public static byte[] getCreationBytes(HttpServletRequest req) throws ParameterException {
    try {
      String creationBytes = req.getParameter(CREATION_BYTES_PARAMETER);
      if(creationBytes == null) {
        // Check the body for the creationBytes as an alternative
        creationBytes = req.getReader().readLine();
        creationBytes = creationBytes.replace("\"", "");
      }
      return Convert.parseHexString(creationBytes);
    } catch (RuntimeException | IOException e) {
      throw new ParameterException(INCORRECT_CREATION_BYTES);
    }
  }
```

 
## Backwards Compatibility
All ATs running after the fork will be limited by the new maximum number of steps.
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
