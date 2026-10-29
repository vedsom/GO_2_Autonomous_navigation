Scenario A — Mapping a brand-new site

Terminal 1 — before starting anything:

bash
ssh unitree@<ROBOT_IP>

If a site is currently loaded in default//graph_map/ and you haven't saved it yet, save it now so you don't lose it:

~/site_manager.sh save <current_site_name>

(Skip this only if you already saved it, or default/ is empty/irrelevant.)

Now start the stack fresh:

export CYCLONEDDS_URI=~/cyclonedds_wifi_test.xml
source /unitree/module/graph_pid_ws/install/setup.bash
bash /unitree/module/graph_pid_ws/0_unitree_slam.sh

Terminal 2:

bash
ssh unitree@<ROBOT_IP>
/unitree/lib/unitree_slam/rocky/build/demo_xt16_replan_manual_goal eth0

Then, at the keyboard prompt:

w                 # start mapping — walk the robot around the new area now
                  # (walk the whole space before continuing)
e                 # end mapping — writes GlobalMap.pcd etc. into default/
x                 # add node 1 (stand at node 1's spot first)
x                 # add node 2 (walk to node 2's spot, then press x)
x                 # ...repeat for every node
c                 # push topology to robot — writes into graph_map/
q                 # close the demo program

Terminal 1 — stop the stack:

Ctrl+C

Terminal 1 (or 2) — save this new site:

bash
~/site_manager.sh save <new_site_name>
~/site_manager.sh list        # sanity check it's there

That site is now permanently archived under ~/unitree_sites/<new_site_name>/.

Scenario B — Returning to a previously saved site

Terminal 1:

ssh unitree@<ROBOT_IP>

Make sure nothing's currently running (if unsure):

~/site_manager.sh load <saved_site_name>
example 
~/site_manager.sh load Test_01
(The script itself will refuse and warn you if it detects the stack still running — so if it errors out, Ctrl+C whatever's running in another session first, then retry the load.)

You should see:

Loading site '<saved_site_name>' into the live PCD + graph paths...
Done. Now start 0_unitree_slam.sh — it will boot with site '<saved_site_name>''s map and graph.

Now start the stack — it boots up already using that site's map and graph:

~/site_manager.sh load hackaton
export CYCLONEDDS_URI=~/cyclonedds_wifi_test.xml
source /unitree/module/graph_pid_ws/install/setup.bash
bash /unitree/module/graph_pid_ws/0_unitree_slam.sh

Terminal 2:

bash
ssh unitree@<ROBOT_IP>
/unitree/lib/unitree_slam/rocky/build/demo_xt16_replan_manual_goal eth0
FOR MULTIPLE GOAL 
/unitree/lib/unitree_slam/rocky/build/demo_xt16_priority_goal eth0

Then:

a                 # relocates on the restored map — no mapping, no node re-entry needed
                  # (waits ~3s to settle, then prompts you)
Enter goal node: 2

...and you're straight into normal goal-driven navigation with the watchdog active.

Quick reference
Situation    Command
Check what's saved    ~/site_manager.sh list
Check what's currently loaded    ~/site_manager.sh current
Save current live map+graph as a site    ~/site_manager.sh save <name>
Restore a saved site as the live one    ~/site_manager.sh load <name>

The one rule that protects everything: never press w (new mapping) without having saved whatever's currently live first — w/e overwrite default/ in place with no backup, and there's no undo once that happens.
