# Space Race - Relativistic Journey

A Phaser 3 space racing game where you pilot a spaceship through a field of asteroids, gravity wells, and relativistic effects to reach Earth in the shortest time possible.

## Features

- **Spaceship Controls**: Rotate and accelerate your ship through space
- **Asteroids**: Obstacles that can be destroyed with ammunition or avoided
- **Gravity Objects**: 
  - **Stars** (yellow): Moderate gravity pull and time dilation
  - **Black Holes** (purple): Strong gravity pull and extreme time dilation
- **Resource Management**:
  - **Fuel**: Required for acceleration
  - **Ammunition**: Used to destroy asteroids
- **Pickups**:
  - **Green stars**: Fuel refills
  - **Orange stars**: Ammunition refills
- **Relativistic Time Mechanics**: 
  - Time passes differently on your ship vs. Earth
  - Proximity to massive objects slows your ship's time
  - High velocity also causes time dilation
  - Goal: Minimize Earth time to reach the destination

## Setup

1. Install dependencies:
```bash
npm install
```

2. Start the development server:
```bash
npm start
```

3. The game will open automatically in your browser at `http://localhost:8080`

## Controls

- **Arrow Keys**: 
  - Left/Right: Rotate ship
  - Up: Accelerate (consumes fuel)
- **Spacebar**: Shoot (consumes ammunition)
- **R**: Restart game (after win/loss)

## Gameplay

1. Start at the left side of the screen
2. Navigate through asteroids, avoiding collisions
3. Use gravity wells strategically (they pull you but also slow your time)
4. Collect fuel and ammo pickups as needed
5. Reach Earth (blue planet on the right) in minimum Earth time
6. Watch the time dilation effects - your ship time vs Earth time!

## Strategy Tips

- Avoid flying too close to gravity objects to minimize time dilation
- Plan your route to balance speed vs. time dilation effects
- Shoot asteroids in your path or navigate around them
- Manage fuel carefully - you need it to accelerate and maneuver
- Collect pickups when safe to do so

## Future Enhancements

- Multiple levels with increasing difficulty
- More complex gravity interactions
- Power-ups and ship upgrades
- Leaderboard for best times
- Enhanced graphics and particle effects
- Sound effects and music
