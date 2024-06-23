require('dotenv').config(): Load environment variables from a .env file.
const express = require('express'): Import the Express module.
const passport = require('passport'): Import the Passport module for authentication.
const GoogleStrategy = require('passport-google-oauth20').Strategy: Import the Google OAuth 2.0 strategy for Passport.
const session = require('express-session'): Import the session middleware.
const { Sequelize, DataTypes } = require('sequelize'): Import Sequelize and DataTypes for ORM.
const Intercom = require('intercom-client'): Import the Intercom client library.
const app = express(): Create an instance of an Express application.
app.use(express.json()): Middleware to parse JSON bodies.
const sequelize = new Sequelize(process.env.MYSQL_URI): Create a new Sequelize instance using the MySQL URI from environment variables.
const User = sequelize.define('User', { ... }): Define the User model with the specified attributes.
const ServiceRequest = sequelize.define('ServiceRequest', { ... }): Define the ServiceRequest model with the specified attributes.
sequelize.sync(): Sync all defined models with the database.
const intercomClient = new Intercom.Client({ token: process.env.INTERCOM_ACCESS_TOKEN }): Create a new Intercom client instance with the access token from environment variables.
passport.use(new GoogleStrategy({ ... }, async (token, tokenSecret, profile, done) => { ... })): Configure Passport to use the Google OAuth 2.0 strategy.
let user = await User.findOne({ where: { googleId: profile.id } }): Find a user by Google ID.
if (!user) { user = await User.create({ ... }) }: Create a new user if one doesn't exist.
passport.serializeUser((user, done) => { done(null, user.id); }): Serialize user information into the session.
passport.deserializeUser(async (id, done) => { const user = await User.findByPk(id); done(null, user); }): Deserialize user information from the session.
app.use(session({ secret: 'secret', resave: false, saveUninitialized: false })): Set up session middleware.
app.use(passport.initialize()): Initialize Passport middleware.
app.use(passport.session()): Enable session support for Passport.
app.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] })): Define a route to start the Google OAuth flow.
app.get('/auth/google/callback', passport.authenticate('google', { failureRedirect: '/' }), (req, res) => { res.redirect('/'); }): Define a route to handle the OAuth callback.
const isAuthenticated = (req, res, next) => { if (req.isAuthenticated()) { return next(); } res.redirect('/auth/google'); }: Middleware to check if the user is authenticated.
app.post('/api/service-request', isAuthenticated, async (req, res) => { ... }): Define a route to submit customer service requests.
const intercomResponse = await intercomClient.messages.create({ ... }): Send the customer service request to Intercom.
const serviceRequest = await ServiceRequest.create({ ... }): Create a new service request record in the database.
app.get('/api/service-requests/
', isAuthenticated, async (req, res) => { ... }): Define a route to retrieve customer service requests by category.
const requests = await ServiceRequest.findAll({ where: { userId: req.user.id, category: req.params.category } }): Fetch service requests from the database.
const PORT = process.env.PORT || 5000: Define the server port.
app.listen(PORT, () => console.log(Server running on port ${PORT})): Start the Express server.

front end:

import React, { useState } from 'react': Import React and useState hook.
import { BrowserRouter as Router, Route, Switch, Redirect } from 'react-router-dom': Import Router components from react-router-dom.
import axios from 'axios': Import axios for making HTTP requests.
import GoogleLogin from 'react-google-login': Import GoogleLogin component.
App Component:

const [user, setUser] = useState(null): State to store user information.
const [requests, setRequests] = useState([]): State to store customer service requests.
const handleLogin = async (response) => { ... }: Function to handle Google login.
const { tokenId } = response: Extract tokenId from response.
const result = await axios.post('/auth/google', { id_token: tokenId }): Send tokenId to backend for authentication.
setUser(result.data): Store user data in state.
const fetchRequests = async (category) => { ... }: Function to fetch customer service requests by category.
const result = await axios.get(/api/service-requests/${category}): Make a GET request to fetch requests.
setRequests(result.data): Store requests in state.
