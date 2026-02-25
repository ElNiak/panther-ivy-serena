---
name: NSCT Methodology
description: This skill should be used when the user asks about "simulation", "Shadow NS", "Shadow Network Simulator", "network simulator", "NSCT", "simulation-based testing", "network topology testing", "large-scale testing", "deterministic testing", or mentions running protocol tests in simulated network environments within the PANTHER framework.
---

# NSCT — Network-Simulator Centric Compositional Testing

## Overview

NSCT is a compositional testing methodology that runs protocol verification in simulated network environments using the Shadow Network Simulator. It enables testing at scale with deterministic execution, complex topologies, and controlled network conditions — complementing NCT's real-network testing.

## Core Concepts

### Shadow Network Simulator Integration
Shadow NS provides deterministic network simulation within PANTHER:
- **Deterministic execution** — Same seed produces identical results, enabling reproducible debugging
- **Scale testing** — Simulate many nodes simultaneously without real hardware
- **Topology control** — Define arbitrary network topologies (meshes, hierarchies, partitions)
- **Network condition modeling** — Simulate latency, packet loss, bandwidth constraints, jitter

### Relationship to NCT and NACT
The three methodologies form a complementary testing strategy:
- **NCT** — Specification compliance in real network conditions
- **NACT** — Security properties under adversarial conditions
- **NSCT** — Behavior at scale and under varied network conditions

All three share the same Ivy formal specification language, 14-layer template, and before/after monitor pattern. The difference is the execution environment and testing focus.

### PANTHER Environment Configuration
NSCT uses PANTHER's experiment configuration with `type: shadow_ns` network environment:

```yaml
tests:
  - name: "NSCT Protocol Test"
    network_environment:
      type: shadow_ns
      topology:
        nodes:
          - name: client_node
            ip: "10.0.0.1"
          - name: server_node
            ip: "10.0.0.2"
        links:
          - source: client_node
            target: server_node
            latency: "50ms"
            bandwidth: "10Mbit"
            loss: "0.1%"
      simulation:
        duration: "60s"
        seed: 42
    services:
      server:
        implementation:
          name: picoquic
          type: iut
        protocol:
          name: quic
          version: rfc9000
          role: server
```

## When to Use NSCT vs NCT

| Criterion | NCT (Real Network) | NSCT (Simulated) |
|---|---|---|
| Fidelity | High (real OS stack) | Medium (simulated stack) |
| Scale | Limited (container resources) | High (many simulated nodes) |
| Determinism | Non-deterministic | Deterministic |
| Topology control | Basic (Docker networks) | Full (arbitrary topologies) |
| Network conditions | Limited manipulation | Full control (latency, loss, bandwidth) |
| Debugging | Harder (non-deterministic) | Easier (deterministic replay) |
| Performance testing | Realistic | Simulated |

**Choose NSCT when:**
- Testing protocol behavior under specific network conditions (latency, loss, bandwidth)
- Testing at scale with many simultaneous nodes
- Need deterministic reproducibility for debugging
- Exploring complex network topologies
- Running regression tests that must produce consistent results

**Choose NCT when:**
- Need realistic network stack behavior
- Testing actual performance characteristics
- Verifying against real-world conditions
- Final validation before deployment

## NSCT Workflow

### Step 1: Define Network Topology
Specify the simulated network graph: nodes, links, latencies, bandwidths, loss rates. Shadow NS supports arbitrary topologies.

### Step 2: Configure Simulation Parameters
Set simulation duration, random seed, logging level. The seed ensures deterministic replay.

### Step 3: Set Up Protocol Implementations
Map IUT implementations to simulated nodes. Each node runs a real protocol implementation within the simulator.

### Step 4: Define Formal Specifications
Reuse the same Ivy specifications from NCT. The formal model is independent of the execution environment.

### Step 5: Write PANTHER Experiment Config
Create a YAML configuration with `type: shadow_ns` network environment. Define topology, simulation parameters, and service mappings.

### Step 6: Execute Simulation
Run via PANTHER:
```bash
panther run --config experiment_config_nsct.yaml
```

### Step 7: Analyze Results
Examine simulation logs and verification output. Shadow NS provides detailed timing information and packet traces.

### Step 8: Iterate with Different Conditions
Modify topology, latency, loss rates, or bandwidth to explore protocol behavior under different conditions. Deterministic seeds allow targeted debugging.

## Shadow NS Build Mode

NSCT requires a specific Z3 build mode for Shadow NS compatibility:
- Use `build_mode: ""` (empty string) in the PANTHER Ivy config
- This uses the legacy `mk_make.py` build system compatible with Shadow NS
- Other build modes (`debug-asan`, `rel-lto`, `release-static-pgo`) are for NCT/NACT Docker environments

## Serena Tools for NSCT

NSCT uses the same panther-serena tools as NCT for specification work:

| Step | Tool | Usage |
|---|---|---|
| Navigate specs | `find_symbol`, `get_symbols_overview` | Understand spec structure |
| Verify specs | `ivy_check` | Check formal properties before simulation |
| Compile tests | `ivy_compile` | Build test executables |
| Create configs | `create_text_file` | Write PANTHER experiment configs |
| Search patterns | `search_for_pattern` | Find relevant configuration patterns |

**IMPORTANT**: Always use panther-serena MCP tools for Ivy operations. Never run ivy_check, ivyc, ivy_show, or ivy_to_cpp directly via Bash.

## Comprehensive Testing Strategy

A complete protocol verification campaign combines all three methodologies:

1. **NCT first** — Verify basic specification compliance with real network
2. **NACT second** — Test resilience against attack scenarios
3. **NSCT third** — Verify behavior at scale and under adverse conditions

Each methodology shares the same Ivy formal specifications but applies them in different execution contexts, providing comprehensive coverage of protocol correctness, security, and robustness.
