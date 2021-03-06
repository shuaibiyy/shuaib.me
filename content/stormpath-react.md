+++
title = "Integrating a React Frontend with Stormpath"
description = ""
date = "2016-02-09T15:35:23+08:00"
+++

In a [previous post](http://shuaib.me/on-stormpath-and-a-java-backend/), I went through how we hooked up Stormpath to our Java backend. In this post, I'll go through how we got it working with our React front-end.

In a [previous post](http://shuaib.me/on-stormpath/), I explained our rationale for not using the Stormpath React SDK:

> For front-end applications, Stormpath provides the Stormpath React SDK, a fairly new project that tries to do a lot of the work required when adding authentication to your app. In our project, however, we opted not to use the SDK. The SDK’s code was fairly small and simple, so we decided it would be more productive in the long run to implement a narrower set of functionalities tailored to our use-case and consistent with our app’s design.

With a REST API backend and a REST client as is our case, it is an established pattern to expect a server response with a payload in a format such as JSON, XML, etc. The Stormpath Servlet Plugin however, responds with paths to server-rendered views for some of its endpoints. This works well if you are rendering your views on the server-side. The URIs to the views returned are [configurable](https://docs.stormpath.com/java/servlet-plugin/registration.html#verify-next-uri), which didn't make much of a difference for us because we prefer containing the view routing logic within the client. Consequently and hopefully temporarily, we ended up with a fire-and-forget model because we couldn't rely on the response from the server, and we didn't want the client to redirect to a server-specified URI. But I digress, the issues we faced with the client-server integration are a topic for another post.

I'll run you through the code required to implement some basic authentication features in a React app using Stormpath. The illustration code is written in ES6, and uses the Reflux flux library. Nonetheless, the code should be easily adaptable to ES5 and/or a different flux library.

First, let's create a login form:

    // login-form.js
    import React from 'react'
	import Reflux from 'reflux'
	import ReactMixin from 'react-mixin'
    import bindAll from 'lodash/bindAll'
	import LinkedStateMixin from 'react-addons-linked-state-mixin'
	import Actions from './actions'	
    import Store from './store'

    @ReactMixin.decorate(LinkedStateMixin)
    @ReactMixin.decorate(Reflux.connect(Store))
    class LoginForm extends React.Component {
    
      constructor (props) {
        super(props)
        bindAll(this)
        this.state = {
          username: '',
          password: ''
        }
      }
       
      static get propTypes () {
	    // History is provided by react-router.
        return {
          history: React.PropTypes.object.isRequired
        }
      }
      
      componentWillUpdate (nextProps, nextState) {
        if (nextState.loggedIn) {
          this.props.history.pushState(null, '/')
        }
      }
    
      onSubmit (e) {
        e.preventDefault()
        const username = this.state.username
        const password = this.state.password
    
        Actions.login({username, password})
      }
    
      render () {
        return (
          <div>
            <form className='login' onSubmit={this.onSubmit.bind(this)}>
              <div className='login-fields'>
                <input
                  type='email'
                  ref='email'
                  className='field'
                  placeholder='Username or Email (required)'
                  valueLink={this.linkState('username')}
                  required='required'
                  />
                <input
                  type='password'
                  className='field'
                  placeholder='Password (required)'
                  valueLink={this.linkState('password')}
                  required='required'
                  />
              </div>
              <input type='submit' className='login-button' value='Log In'/>
            </form>
            <LoginInfo/>
          </div>
        )
      }
    }
    
    export default LoginForm

The component above uses the [react-mixin](https://github.com/brigand/react-mixin) library to make decorators work with React ES6 components. You'll need to use Babel 5.x or a [plugin](https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy) if you are on Babel 6, because the unratified decorator syntax is no longer supported.

Next we'll create a registration form:

    // registration-form.js
    import React from 'react'
    import Reflux from 'reflux'
    import bindAll from 'lodash/bindAll'
    import ReactMixin from 'react-mixin'
    import LinkedStateMixin from 'react-addons-linked-state-mixin'
    import Actions from './actions'
    import Store from './store'
    
    @ReactMixin.decorate(LinkedStateMixin)
    @ReactMixin.decorate(Reflux.connect(Store))
    class RegistrationForm extends React.Component {
    
      constructor (props, context) {
        super(props, context)
        bindAll(this)
        this.state = {
          givenName: '',
          surname: '',
          username: '',
          password: '',
          confirmPassword: '',
          errorMessage: null
        }
      }
    
      static get propTypes () {
        return {
          history: React.PropTypes.object.isRequired
        }
      }
    
      componentWillUpdate (nextProps, nextState) {
        if (nextState.newlyRegistered) {
          this.props.history.pushState(null, '/')
        }
      }
    
      onSubmit (e) {
        e.preventDefault()
        const givenName = this.state.givenName
        const surname = this.state.surname
        const email = this.state.email
        const password = this.state.password
        const confirmPassword = this.state.confirmPassword
    
        Actions.register({
          givenName,
          surname,
          email,
          password,
          confirmPassword
        })
      }
    
      onGivenNameChanged (e) {
        this.state.givenName = e.target.value
      }
    
      onSurnameChanged (e) {
        this.state.surname = e.target.value
      }
    
      onEmailChanged (e) {
        this.state.email = e.target.value
      }
    
      onPasswordChanged (e) {
        this.state.password = e.target.value
      }
    
      onConfirmPasswordChanged (e) {
        this.state.confirmPassword = e.target.value
    
        if (this.state.password !== this.state.confirmPassword) {
          this.setState({errorMessage: 'Passwords don't match.'})
        } else {
          this.setState({errorMessage: null})
        }
      }
    
      render () {
        return (
          <div>
            <form className='login' onSubmit={this.onSubmit.bind(this)}>
              <div className='login-fields'>
                <input
                  type='text'
                  ref='givenName'
                  className='field'
                  placeholder='Given Name (required)'
                  onChange={this.onGivenNameChanged.bind(this)}
                  required='required'
                  />
                <input
                  type='text'
                  className='field'
                  placeholder='Surname (required)'
                  onChange={this.onSurnameChanged.bind(this)}
                  required='required'
                  />
                <input
                  type='email'
                  className='field'
                  placeholder='Email (required)'
                  onChange={this.onEmailChanged.bind(this)}
                  required='required'
                  />
                <input
                  type='password'
                  className='field'
                  placeholder='Password (required)'
                  onChange={this.onPasswordChanged.bind(this)}
                  required='required'
                  />
                <input
                  type='password'
                  className='field'
                  placeholder='Confirm Password (required)'
                  onChange={this.onConfirmPasswordChanged.bind(this)}
                  required='required'
                  />
              </div>
              {
                this.state.errorMessage !== null
                  ? <p style={{color: 'red'}}>{this.state.errorMessage}</p>
                  : null
              }
              <input type='submit' className='login-button' value='Create Account'/>
          </div>
        )
      }
    }
    
    export default RegistrationForm

Next, we create the actions that get fired when forms are submitted:

    // actions.js
    import Reflux from 'reflux'
    import { login, register } from './endpoints'
    
    const Actions = Reflux.createActions({
      login: {children: ['completed', 'failed']},
      register: {children: ['completed']},
    })
    Actions.login.listen(function (credentials) {
      return login(credentials).then(this.completed).catch(this.failed)
    })
    Actions.register.listen(function (credentials) {
      this.progressed()
      // Register returns a view, not a JSON response.
      return register(credentials).then(this.completed).catch(this.completed)
    })
    
    export default Actions

We use the [Fetch API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) to make calls to our API using [isomorphic fetch](https://github.com/matthew-andrews/isomorphic-fetch), but you can easily replace it with XHR or any of its wrapper libraries:

    // endpoints.js
    import fetch from 'isomorphic-fetch'
    
    const API_URL = 'http://localhost:8080/api'
    
    export function login (credentials) {
      return fetch(`${API_URL}/oauth/token`, {
        method: 'post',
        credentials: 'include',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: `grant_type=password&username=${credentials.username}&password=${credentials.password}`
      })
    }
    
    export function register (credentials) {
      return fetch(`${API_URL}/register`, {
        method: 'post',
        credentials: 'include',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: `givenName=${credentials.givenName}&surname=${credentials.surname}&email=${credentials.email}&password=${credentials.password}&confirmPassword=${credentials.confirmPassword}`
      })
    }
    
You'll notice that I construct the request body, as in: `grant_type=password&username=${credentials.username}&password=${credentials.password}`. I do so because for reasons I've yet to explore, the Stormpath endpoints don't like bodies formed using FormData, as in:

    let formData = new FormData()
    formData.append("username", credentials.username)
    formData.append("password", credentials.password)
    formData.append("grant_type", "password")

The final step is just a matter of listening for actions in your store, and triggering state changes in your components.

    //store.js
    import Reflux from 'reflux'
    import Actions from '../actions'
    
    const Store = Reflux.createStore({
      listenables: [Actions],
      
      onLoginCompleted: function (response) {
        if (response.ok) {
          this.trigger({loggedIn: true})
        }
      },
    
      onRegisterCompleted: function () {
        this.trigger({newlyRegistered: true})
      }
    })
    
    export default Store

That's it.

Unrelated to React and Stormpath, I ran into a number of issues while using the Fetch API, and here are my tips for saving yourself some valuable time:

 1. The [Fetch Standard](https://fetch.spec.whatwg.org/) is your friend. 
 2. `Credentials: “include”` is important.
 3. Understand the difference between `cors` and `no-cors`.