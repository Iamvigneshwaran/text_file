# App.js //client side --react 


import React, { useEffect, useState } from 'react';
import axios from 'axios';
import { BrowserRouter, Routes, Route } from "react-router-dom";
import User from './components/User';

const UserList = () => {
  const [users, setUsers] = useState([]);
  const [id, setId] =useState()
  const [name, setName] = useState('');
  const [age, setAge] = useState('');

  useEffect(() => {
    axios.get('http://localhost:5000/users')
      .then(res => {
        setUsers(res.data);
      })
      .catch(err => {
        console.error(err);
      })
  }, []);

  const handleSubmit = (event) => {
    event.preventDefault();
    const newUser = { id,name, age };
    
    axios.post('http://localhost:5000/users', newUser)
      .then(res => {
        setUsers([...users, res.data]);
        setId(id+1)
        setId("")
        setName('');
        setAge('');
      })
      .catch(err => {
        console.error(err);
      })
  }

  return (
    <div>
      <>
              <h2>User List</h2>
              <table>
                <thead>
                  <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Age</th>
                  </tr>
                </thead>
                <tbody>
                  {users.map(user => (
                    <tr key={user._id}>
                      <td>{user.id}</td>
                      <td>{user.name}</td>
                      <td>{user.age}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
              </>
          
      <BrowserRouter>
        <Routes>
          <Route path='/user' element={<> <form onSubmit={handleSubmit}>
          <label>
                  Id :
                  <input type="text" value={id} onChange={e => setId(e.target.value)} />
                </label> <br></br>
          
                <label>
                  Name:
                  <input type="text" value={name} onChange={e => setName(e.target.value)} />
                </label> <br></br>
                <label>
                  Age:
                  <input type="text" value={age} onChange={e => setAge(e.target.value)} />
                </label><br></br>
                <button type="submit">Add User</button>
              </form></>}/>
          
            
             
         
        </Routes>
      </BrowserRouter>
    </div>
  );
}

export default UserList;
<!---------------------------------------------------------------------------------------------------------------------------------------------------------------->
//server side app.js
const express = require("express");
const mongoose = require("mongoose");
const bodyParser = require("body-parser");
const cors = require('cors'); 

const app = express();
app.use(cors());

const urlEncoder = bodyParser.urlencoded({ extended: false });
app.use(urlEncoder)
app.use(express.json())

mongoose.connect("mongodb://127.0.0.1:27017/newCollection", {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => {
  console.log("MongoDB started...");
}).catch(console.log);

const userSchema =  new mongoose.Schema({
  id:Number,
  name : String,
  age:Number
});

const userModel = mongoose.model('newusers', userSchema);

app.get("/users", async (req, res) => {
  try {
    const users = await userModel.find();
    res.send(users);
  } catch (err) {
    console.error(err);
    res.status(500).send(err);
  }
});

app.post('/users', async (req, res) => {
  try {
    const user = new userModel(req.body);
    console.log(user)
    await user.save();
    res.status(201).send(user);
  } catch (err) {
    console.error(err);
    res.status(500).send(err);
  }
});

app.put('/users/:id', async (req, res) => {
  try {
    const user = await userModel.findByIdAndUpdate(req.params.id, req.body);
    await user.save();
    res.status(200).send(user);
  } catch (err) {
    console.error(err);
    res.status(500).send(err);  
  }
});

app.delete('/users/:id', async (req, res) => {
  try {
    const user = await userModel.findByIdAndDelete(req.params.id);
    if (!user) res.status(404).send("No user found");
    res.status(200).send(user);
  } catch (err) {
    console.error(err);
    res.status(500).send(err);
  }
});

app.listen(5000, () => {
  console.log("App is running on Port 5000");
});
