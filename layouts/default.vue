<template>
	<div class="body">
		<transition name="fade">
			<nuxt class="content" />
		</transition>
		<footer v-if="shouldShowNavbar" class="navbar">
			<nuxt-link to="/" class="nav-element">
				<div class="nav-item">
					<div class="nav-icon">
						<b-icon pack="fas" icon="comments" />
					</div>
					<div class="nav-caption">Moments</div>
				</div>
			</nuxt-link>
			<nuxt-link to="/bot" class="nav-element">
				<div class="nav-item">
					<div class="nav-icon">
						<b-icon pack="fas" icon="robot" />
					</div>
					<div class="nav-caption">CLS Bot</div>
				</div>
			</nuxt-link>
			<nuxt-link to="/explore" class="nav-element">
				<div class="nav-item">
					<div class="nav-icon">
						<b-icon pack="fas" icon="compass" />
					</div>
					<div class="nav-caption">Explore</div>
				</div>
			</nuxt-link>
			<a class="nav-element" @click.prevent="a11y">
				<div class="nav-item">
					<div class="nav-icon">
						<b-icon pack="fas" icon="universal-access" />
					</div>
					<div class="nav-caption">Access</div>
				</div>
			</a>
		</footer>
	</div>
</template>

<script>
import firebase from "@/services/firebase";
import firestore from "@/services/firestore";
import "@/services/registerServiceWorker";

import loadJs from "loadjs";

import { LOGIN_SUCCESS } from '@/store/user';

export default {
	mounted() {
		window.a11ySettings = window.a11ySettings || {};
		// window.a11ySettings.display = "none";
		loadJs("https://agastya-loader.oswaldlabs.com/hackathon.js", () => {
			setTimeout(() => {
				window.agastya.api();
			}, 1000);
        });

        firebase.auth().onAuthStateChanged(userProfile => {
            if (userProfile) {
                this.$store.commit(LOGIN_SUCCESS, userProfile);

                if (this.$route.path === "/login" || this.$route.path === "/loginui") {
                    this.$router.push("/");
                }

                firestore.collection('users').doc(userProfile.uid).set({
                    createdAt: new Date(),
                    updatedAt: new Date(),
                    creationTime: userProfile.metadata.creationTime,
                    lastSignInTime: userProfile.metadata.lastSignInTime,
                    name: userProfile.displayName,
                    email: userProfile.email,
                    photoUrl: userProfile.photoURL
                }, { merge: true });

                this.askForPermissioToReceiveNotifications();
            }
        });
	},
	methods: {
		a11y() {
			window.agastya.frame.open();
        },
        async askForPermissioToReceiveNotifications() {
            try {
                const messaging = firebase.messaging();
                await messaging.requestPermission();
                const token = await messaging.getToken();
                console.log('Notifications token', token);
                if (token) this.saveMessagingToken(token);
            } catch (error) {
                console.log(error);
            }
        },
        saveMessagingToken(token) {
            if (!this.$store.state.user.loggedIn) return;
            firestore.collection('users').doc(this.$store.state.user.profile.uid).set({
                messagingToken: token
            }, { merge: true });
        }
    },
    computed: {
        shouldShowNavbar() {
            return !['/login'].includes(this.$route.path);
        }
    }
}
</script>


<style lang="scss">
@import url("https://use.fontawesome.com/releases/v5.5.0/css/all.css");
@import url("https://fonts.googleapis.com/css?family=Lato:400,700");
body {
	font-family: "Lato";
}
html, body {
	background-color: #dfe6e9;
}
header, footer.navbar {
	box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
	background-color: #fff;
	z-index: 5;
	position: fixed;
	left: 0; right: 0;
}
header {
	font-size: 125%;
	top: 0;
	height: 4rem;
	line-height: 4rem;
	text-align: center;
}
h2 {
	font-size: 95% !important;
	text-transform: uppercase;
	margin-bottom: 1rem;
	font-weight: 700 !important;
}
#agastyabutton {
	opacity: 0 !important;
}
main {
	padding: 4rem 0;
}
footer.navbar {
	bottom: 0;
	justify-content: space-between;
	align-items: stretch;
	display: flex;
	.nav-element {
		color: inherit;
		appearance: none;
		height: 4rem;
		background: transparent;
		border: none;
		font: inherit;
		display: flex;
		flex-direction: column;
		border-radius: 0;
		padding: 0;
		margin: 0;
		width: 100%;
		justify-content: center;
		text-align: center;
		.nav-item {
			align-self: center;
			display: flex;
			justify-content: center;
			flex-direction: column;
		}
		.nav-caption {
			font-size: 80%;
		}
		.nav-icon {
			margin: 0.15rem auto;
		}
		&.nuxt-link-exact-active {
			color: #1d3176;
			.nav-icon {
				transform: scale(1.2);
			}
		}
	}
}
.fade-enter-active, .fade-leave-active {
	transition: opacity 0.2s;
}
.fade-enter, .fade-leave-to {
	opacity: 0;
}
.empty-state {
	padding: 20vh 0;
	text-align: center;
	font-size: 125%;
	h2 {
		font-size: 150%;
		font-weight: 300;
	}
}
.groups {
	display: flex;
	flex-wrap: wrap;
	li {
		display: block;
		flex: 0 0 50%;
    	max-width: 50%;
		padding: 0 1rem;
		&:nth-child(2n) {
			padding-left: 0.5rem;
		}
		&:nth-child(2n - 1) {
			padding-right: 0.5rem;
		}
	}
	a {
		display: block;
		width: 100%;
		box-shadow: 0px 3px 7px rgba(0, 0, 0, 0.1);
		border-radius: 0.25rem;
		overflow: hidden;
		margin-bottom: 1rem;
		img {
			border-radius: 0;
			margin-left: 0;
			display: block;
			width: 100%;
			padding: 0;
			height: auto;
		}
		div {
			padding: 1rem;
		}
	}
}
/* Medium devices (landscape tablets, 768px and up) */
@media only screen and (min-width: 768px) {
    .groups {
        li {
            flex: 0 0 25%;
        }
    }
}
ul {
	margin: 0 !important;
	padding: 0 !important;
	list-style: none !important;
}
li {
	margin-top: 0 !important;
}
li a {
	display: block;
	background-color: #fff;
	color: inherit;
	border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}
</style>
