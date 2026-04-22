"use client";

import { useEffect, useRef } from "react";
import * as THREE from "three";

export default function FlightSimulator() {
  const canvasRef = useRef(null);
  const mountRef = useRef(null);

  useEffect(() => {
    if (!canvasRef.current || !mountRef.current) return;

    // Scene setup
    const scene = new THREE.Scene();
    scene.fog = new THREE.Fog(0x87CEEB, 100, 10000); // Light blue fog

    // Camera
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      10000
    );
    camera.position.set(0, 20, 50);

    // Renderer
    const renderer = new THREE.WebGLRenderer({ 
      canvas: canvasRef.current,
      antialias: true 
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setClearColor(0x87CEEB); // Sky blue background

    // Skybox (procedural gradient)
    const skyGeometry = new THREE.SphereGeometry(5000, 32, 32);
    const skyMaterial = new THREE.ShaderMaterial({
      vertexShader: `
        varying vec3 vWorldPosition;
        void main() {
          vec4 worldPosition = modelMatrix * vec4(position, 1.0);
          vWorldPosition = worldPosition.xyz;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `,
      fragmentShader: `
        varying vec3 vWorldPosition;
        void main() {
          vec3 dir = normalize(vWorldPosition);
          float t = 0.5 * (dir.y + 1.0);
          vec3 skyColor = mix(vec3(0.53, 0.81, 0.98), vec3(0.8, 0.9, 1.0), t);
          gl_FragColor = vec4(skyColor, 1.0);
        }
      `,
      side: THREE.BackSide
    });
    const skybox = new THREE.Mesh(skyGeometry, skyMaterial);
    scene.add(skybox);

    // Ocean plane
    const oceanGeometry = new THREE.PlaneGeometry(10000, 10000);
    const oceanMaterial = new THREE.MeshLambertMaterial({ 
      color: 0x006994,
      transparent: true,
      opacity: 0.8
    });
    const ocean = new THREE.Mesh(oceanGeometry, oceanMaterial);
    ocean.rotation.x = -Math.PI / 2;
    ocean.position.y = -50;
    scene.add(ocean);

    // Grid helper for speed reference
    const gridHelper = new THREE.GridHelper(1000, 50, 0x444444, 0x222222);
    gridHelper.position.y = -49;
    scene.add(gridHelper);

    // Lighting
    const ambientLight = new THREE.AmbientLight(0x404040, 0.6);
    scene.add(ambientLight);
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
    directionalLight.position.set(100, 100, 50);
    scene.add(directionalLight);

    // Airplane model (simplified white plane with red tail)
    const planeGroup = new THREE.Group();
    
    // Fuselage
    const fuselageGeometry = new THREE.CylinderGeometry(1, 1.2, 12, 8);
    const fuselageMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff });
    const fuselage = new THREE.Mesh(fuselageGeometry, fuselageMaterial);
    fuselage.rotation.z = Math.PI / 2;
    planeGroup.add(fuselage);

    // Wings
    const wingsGeometry = new THREE.BoxGeometry(20, 0.3, 4);
    const wingsMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff });
    const wings = new THREE.Mesh(wingsGeometry, wingsMaterial);
    wings.position.y = 0.5;
    planeGroup.add(wings);

    // Tail (red)
    const tailGeometry = new THREE.BoxGeometry(6, 3, 0.3);
    const tailMaterial = new THREE.MeshLambertMaterial({ color: 0xff0000 });
    const tail = new THREE.Mesh(tailGeometry, tailMaterial);
    tail.position.set(-6, 1.5, 0);
    planeGroup.add(tail);

    // Vertical stabilizer
    const vStabGeometry = new THREE.BoxGeometry(0.3, 4, 3);
    const vStab = new THREE.Mesh(vStabGeometry, tailMaterial);
    vStab.position.set(-8, 2, 0);
    planeGroup.add(vStab);

    planeGroup.position.y = 20;
    scene.add(planeGroup);

    // Physics variables
    let velocity = new THREE.Vector3(0, 0, 100); // Forward speed
    let pitch = 0;
    let roll = 0;
    let throttle = 0.5;
    let altitude = 20;

    // Controls
    const keys = {};
    const joystick = { x: 0, y: 0 };
    const touchControls = {};

    // Mobile Controls
    let leftJoystickActive = false;
    let rightSliderActive = false;
    let joystickCenter = { x: 100, y: window.innerHeight - 150 };
    let sliderPos = { y: window.innerHeight - 100 };

    // Event listeners
    const handleResize = () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    };

    const handleKeyDown = (e) => {
      keys[e.code] = true;
    };

    const handleKeyUp = (e) => {
      keys[e.code] = false;
    };

    const handleTouchStart = (e) => {
      e.preventDefault();
      for (let touch of e.changedTouches) {
        const rect = canvasRef.current.getBoundingClientRect();
        const x = touch.clientX - rect.left;
        const y = touch.clientY - rect.top;

        if (x < window.innerWidth / 2) {
          // Left joystick area
          leftJoystickActive = true;
          joystickCenter = { x, y };
        } else {
          // Right slider area
          rightSliderActive = true;
          sliderPos.y = y;
          throttle = 1.0 - (y - 50) / (window.innerHeight - 100);
          throttle = Math.max(0, Math.min(1, throttle));
        }
      }
    };

    const handleTouchMove = (e) => {
      e.preventDefault();
      for (let touch of e.changedTouches) {
        const rect = canvasRef.current.getBoundingClientRect();
        const x = touch.clientX - rect.left;
        const y = touch.clientY - rect.top;

        if (leftJoystickActive) {
          const deltaX = x - joystickCenter.x;
          const deltaY = y - joystickCenter.y;
          const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
          const maxDistance = 80;
          
          if (distance < maxDistance) {
            joystick.x = deltaX / maxDistance;
            joystick.y = deltaY / maxDistance;
          } else {
            joystick.x = (deltaX / distance) * 0.8;
            joystick.y = (deltaY / distance) * 0.8;
          }
        } else if (rightSliderActive) {
          sliderPos.y = Math.max(50, Math.min(window.innerHeight - 50, y));
          throttle = 1.0 - (sliderPos.y - 50) / (window.innerHeight - 100);
          throttle = Math.max(0, Math.min(1, throttle));
        }
      }
    };

    const handleTouchEnd = (e) => {
      e.preventDefault();
      for (let touch of e.changedTouches) {
        if (leftJoystickActive) {
          leftJoystickActive = false;
          joystick.x = 0;
          joystick.y = 0;
        } else if (rightSliderActive) {
          rightSliderActive = false;
        }
      }
    };

    window.addEventListener("resize", handleResize);
    window.addEventListener("keydown", handleKeyDown);
    window.addEventListener("keyup", handleKeyUp);
    canvasRef.current.addEventListener("touchstart", handleTouchStart);
    canvasRef.current.addEventListener("touchmove", handleTouchMove);
    canvasRef.current.addEventListener("touchend", handleTouchEnd);

    // Animation loop
    const animate = () => {
      requestAnimationFrame(animate);

      // Input processing
      const pitchInput = joystick.y * 0.02 + (keys["ArrowUp"] ? 0.02 : 0) - (keys["ArrowDown"] ? 0.02 : 0);
      const rollInput = joystick.x * 0.02 + (keys["ArrowLeft"] ? 0.02 : 0) - (keys["ArrowRight"] ? 0.02 : 0);

      // Smooth pitch & roll (CRITICAL: Clamp pitch to prevent gimbal lock)
      pitch += pitchInput * 0.1;
      roll += rollInput * 0.1;
      pitch *= 0.92; // Damping
      roll *= 0.92;
      
      // CLAMP PITCH -0.5 to 0.5 radians (CRITICAL)
      pitch = Math.max(-0.5, Math.min(0.5, pitch));

      // Update plane physics
      const speed = 80 + throttle * 120;
      velocity.z = speed;

      // Apply rotations
      planeGroup.rotation.x = pitch;
      planeGroup.rotation.z = roll;

      // Forward movement
      planeGroup.position.add(velocity.clone().applyQuaternion(planeGroup.quaternion));
      
      // Altitude maintenance
      altitude += Math.sin(pitch) * 0.5;
      planeGroup.position.y = Math.max(10, altitude);

      // Ocean wave effect
      ocean.rotation.z += 0.001;

      // Third-person camera (rigid offset behind and above)
      const idealOffset = new THREE.Vector3(0, 15, 40);
      idealOffset.applyQuaternion(planeGroup.quaternion);
      camera.position.copy(planeGroup.position).add(idealOffset);
      camera.lookAt(planeGroup.position); // CRITICAL: Every frame

      renderer.render(scene, camera);
    };

    animate();

    // Cleanup
    return () => {
      window.removeEventListener("resize", handleResize);
      window.removeEventListener("keydown", handleKeyDown);
      window.removeEventListener("keyup", handleKeyUp);
      
      if (canvasRef.current) {
        canvasRef.current.removeEventListener("touchstart", handleTouchStart);
        canvasRef.current.removeEventListener("touchmove", handleTouchMove);
        canvasRef.current.removeEventListener("touchend", handleTouchEnd);
      }

      renderer.dispose();
      mountRef.current = null;
    };
  }, []);

  return (
    <div 
      ref={mountRef}
      style={{ 
        width: "100vw", 
        height: "100vh", 
        position: "relative",
        background: "linear-gradient(to bottom, #87CEEB 0%, #98D8E8 100%)"
      }}
    >
      <canvas 
        ref={canvasRef}
        style={{
          display: "block",
          width: "100%",
          height: "100%",
          touchAction: "none"
        }}
      />
      
      {/* Mobile Control Instructions */}
      <div style={{
        position: "absolute",
        top: 20,
        left: 20,
        color: "white",
        fontFamily: "Arial, sans-serif",
        fontSize: 14,
        textShadow: "2px 2px 4px rgba(0,0,0,0.8)",
        zIndex: 100
      }}>
        <div>← → Pitch/Roll | ↑ ↓ Throttle</div>
        <div>Touch left: Joystick | Touch right: Throttle</div>
      </div>
    </div>
  );
}
