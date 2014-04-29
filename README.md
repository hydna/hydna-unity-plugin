##Quickstart with hydna for Unity##

Our Unity plugin works with Unity Free as well as Unity Pro on all platforms. Visit https://www.hydna.com/documentation for full spec of our API. This plugin is based on the hydna .NET bindings, see https://github.com/hydna/Hydna.NET for more info.

###Step by step###

1. Create a free account at https://www.hydna.com/account/create/ and then create yourself a free hydna domain.

2. Place the **Hydna.Net.dll** in your projects plugin folder **Yourprojectname**/Assets/Plugins

3. In your project create an empty gameobject and add a new script to it called **HydnaChat** and choose CSharp as the language.

4. Replace the stub code in your newly created **HydnaChat** script with the below script also found in **HydnaChat.cs**.

5. Finally replace "**yourdomain.hydna.net**" with the domain you created in step 1. 

C# example:

    using UnityEngine;
    using System.Collections;
    using Hydna.Net;

    public class HydnaChat : MonoBehaviour {

        public bool show = false;

        private Channel conn;
        private bool hasSentEnter = false;
        private string input_txt = "";
        private string message_buffer = "";

        private Vector2 scrollPosition;

        void Start () {

            conn = new Channel();

            conn.Connect ("yourdomain.hydna.net", ChannelMode.ReadWrite);

            conn.Open += delegate(object sender, ChannelEventArgs e) {

                message_buffer += "* Connected to channel";
            };

            conn.Data += delegate(object sender, ChannelDataEventArgs e) {

                message_buffer += "\n" + e.Text;

                scrollPosition.y = Mathf.Infinity;

            };

            conn.Closed += delegate(object sender, ChannelCloseEventArgs e) {

                message_buffer += "\n * Disconnected from channel";
            };

        }

        void Send (){

            if(conn.State == ChannelState.Open && input_txt.Length > 0){
                conn.Send (input_txt);
            }

            input_txt = "";
        }

        void Toggle(){
            if (show) {
                show = false;
            } else {
                show = true;
                scrollPosition.y = Mathf.Infinity;
            }
        }

        void OnGUI (){

            int window_w = Screen.width; 
            int window_h = Screen.height;
        
            int padding = 20;
            int margin = 10;

            int box_width = window_w - (padding * 2);
            int box_height = window_h - (padding * 2);

            int submit_width = 80;
            int submit_height = 24;

            int input_width = box_width - ((margin * 2) + submit_width + margin);
            int input_height = submit_height;

            int chat_width = box_width - (margin * 2);
            int chat_height = box_height - (submit_height + (padding * 2) + (margin * 2));

            if (show) {

                GUI.Box (new Rect (padding, padding, box_width, box_height), "Let's have a chat");

                GUILayout.BeginArea(new Rect(padding+margin, (padding * 2) + (margin * 2), chat_width, chat_height));
                scrollPosition = GUILayout.BeginScrollView(scrollPosition);
                GUILayout.Label (message_buffer);

                GUILayout.EndScrollView();
                GUILayout.EndArea();

                Event e = Event.current;        
                if (e.type == EventType.KeyDown && e.keyCode == KeyCode.Return) {       
                    if(!hasSentEnter){
                        Send();
                        hasSentEnter = true;
                    }
                }

                if (e.type == EventType.KeyUp && e.keyCode == KeyCode.Return) {  
                    hasSentEnter = false;
                }

                input_txt = GUI.TextField (new Rect (padding + margin, window_h - (padding + margin + input_height), input_width, input_height), input_txt, 200);

                if(GUI.Button (new Rect (input_width + padding + (margin * 2), window_h - (padding + margin + input_height), submit_width, submit_height), "Send")) {

                    Send ();
                }

                if(GUI.Button (new Rect (padding+margin, padding+margin, submit_height, submit_height), "x")) {
                
                    Toggle ();
                }

            }else{

                if(GUI.Button (new Rect (padding, padding, submit_width, submit_height), "Chat")) {
                
                    Toggle ();
                }
            }
        }
    }
