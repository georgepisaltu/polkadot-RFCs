# RFC-0136: Election mechanism for invulnerable collators on system chains

|                 |                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------- |
| **Start Date**  | 29 January 2025                                                                             |
| **Description** | Mechanism for electing invulnerable collators on system chains.                             |
| **Authors**     | George Pisaltu                                                                              |

## Summary

The current election mechanism for permissionless collators on system chains was introduced in RFC-7. This RFC proposes a mechanism to facilitate replacements in the invulnerable sets of system chains by breaking down barriers that exist today.

## Motivation

Following RFC-7 and the [introduction of the collator election mechanism](https://github.com/paritytech/polkadot-sdk/pull/1340), anyone can now collate on a system chain on the permissionless slots, but the invulnerable set has been a contentious issue among current collators on system chains as the path towards an invulnerable slot is almost impossible to pursue. From a technical standpoint, nothing is preventing a permissionless collator, or anyone for that matter, from submitting a referendum to remove one collator from the invulnerable set and add themselves in their place. However, as it quickly becomes obvious, such a referendum would be very difficult to pass under normal circumstances.

The first reason this would be contentious is that there is no significant difference between collators with good performance. There is no reasonable way to keep track of arbitrary data on-chain which could clearly and consistently distinguish between one collator or another. Collators that perform well propose blocks when they are supposed to and that is what is being tracked on-chain. Any other metrics for performance are arbitrary as far as the runtime logic is concerned and should be reasoned upon by humans using public discussion and a referendum.

The second reason for this is the inherently social aspect of this action. Even just proposing the referendum would be perceived as an attack on a specific collator in the set, singling them out, when in reality the proposer likely just wants to be part of the set and doesn't necessarily care who is kicked. In order to consolidate their position, the other invulnerables will rally behind the one that was challenged and the bid to replace one invulnerable will probably fail.

Existing invulnerables have a vested interest in protecting any other invulnerable from such attacks so that they themselves would be protected if need be. The existing collator set has already demonstrated that they can work together and subvert the free market mechanism offered by the runtime when they agreed to not outbid each other on permissionless slots after the new collator selection mechanism was introduced.

The existing invulnerable set on a given system chain are there for a reason; they have demonstrated reliability in the past and were rewarded by governance with invulnerable slots and a bounty to cover their expenses. This means they have a solid reputation and a strong say in governance over matters related to collation. The optics of a permissionless collator actively challenging an invulnerable, even when it's justified, combined with the support of other invulnerables, make the invulnerable set de facto immutable. While there should be strong guarantees of stability for invulnerables, they should not be a closed circle.

## Stakeholders

- Infrastructure providers (people who run validator/collator nodes)
- Polkadot Treasury

## Explanation

### Proposal

This RFC proposes a periodic, mandatory, round-robin, first past the post election mechanism for invulnerables.

### How it works

The election should be implemented on top of the current logic in the `collator-selection` pallet. In this mechanism, candidates would register for the next election by placing deposits. When the period between elections passes, the election starts with every candidate that registered as an option on the ballot. After a certain amount of time passes, the election closes and the candidate with the most votes, expressed as locked tokens, wins the election and claims the invulnerable spot. The period between elections should be configurable at the `collator-selection` pallet level. A full election cycle ends when the pallet held an election for every single invulnerable slot. To qualify for the ballot, candidates must have been collating for at least one period from a permissionless slot or be the incumbent.

### Motivations behind the particularities of this mechanism

- Round-robin - It is not desirable to allow any election of the entire invulnerable set at once because the main purpose of invulnerables is to ensure the stability, reliability and liveness of the parachain. It is safer to change them one by one and, in case mistakes happen, governance has time to react without endangering the liveness of any chain.
- First past the post - It is the simplest election mechanism for one position with multiple candidates.
- Mandatory - The runtime, not any particular individual, is actively pushing the invulnerables to convince people that they not only deserve to keep their invulnerable slots, but that they deserve it more than any of the other candidates that registered; the rules of the chain enforce this mechanism so no blame or ill-intent can be attributed to other individuals.
- Periodic - In order to provide a reasonable path towards an invulnerable slot, no seat can be permanent and should be challenged periodically.
- Ballot qualification - Any invulnerable collator must have a proven track record as a collator, so allowing only current permissionless collators to run against the current invulnerable minimizes the chance of human error by restricting the number of incompatible choices.

### Corner cases

- If no candidate registers for an election, the slot will become empty, unless the number of collators is lower than the minimum number allowed by the pallet configuration, defined in `MinEligibleCollators`.
- In case of equality for the first and second positions, the candidate that registered first wins the election.

## Drawbacks

TODO

## Testing, Security, and Privacy

All election mechanisms as well as corner cases can be covered with unit tests.

## Performance, Ergonomics, and Compatibility

### Performance

The chain will have to run extrinsics to start and end elections periodically, but the impact in terms of weight and PoV size is negligible.

### Ergonomics

The invulnerables will be the most affected group, as they will have to now compete in elections periodically to secure their spots. Permissionless candidates will now have a clear, though not guaranteed, path towards becoming an invulnerable, at least for a period of time.

### Compatibility

Any changes to the election mechanism of invulnerables should be compatible with the current invulnerable set interaction with the collator set chosen at the session boundary. The current invulnerable set for each chain can be grandfathered in when upgrading the `collator-selection` pallet version.

## Prior Art and References

This RFC builds on RFC-7, which introduced the election mechanism for system chain collators.

## Unresolved Questions

- How long should the period between individual elections be? How long should the full election cycle be? A tentative answer to this would be that there should be a bit more than one month between individual elections, so that if there are 5 invulnerables on system chains, a full election cycle would take 6 months.
- How long should the voting stay open?
- For how long should the tokens of the winning vote be locked?
- Should we allow vote delegation or only direct voting?

## Future Directions and Related Material

The main spinoff of this RFC might be a multiple choice poll implementation in a separate pallet which, as far as I'm aware, doesn't exist at this point in time. Additionally, a more complex solution where the voting for all system chains happens in a single place which then sends XCM responses with election results back to system chains can be implemented in the next iteration of this RFC.
