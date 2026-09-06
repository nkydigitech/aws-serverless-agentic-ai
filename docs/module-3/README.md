# Module 3: Hotel Recommendation Agent (Extension)

## Overview
Optional extension - Adding a Hotel Recommendation Agent to the choreography pattern.

This module extends Module 1 by adding a 4th agent that reacts to `FlightSearchCompleted` events.

**Status:** Planned - Part of 15-Module Ansible Lab series

## Architecture
EventBridge fan-out makes it easy - just add new Rule + Target, no touching existing agents.

## Coming Soon
- Hotel Agent Lambda
- EventBridge Rule for hotel search
- Integration with Module 1 bus
