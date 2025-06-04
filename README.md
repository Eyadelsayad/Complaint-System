# Complaint-System
import { useState } from 'react'
import { Button } from "/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle, CardDescription, CardFooter } from "/components/ui/card"
import { Input } from "/components/ui/input"
import { Textarea } from "/components/ui/textarea"
import { Label } from "/components/ui/label"
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "/components/ui/select"
import { Shield, User, ArrowLeft, Mail, Search } from "lucide-react"

type Complaint = {
  id: string  // Will store numeric string
  name: string
  userId: string
  description: string
  status: 'pending' | 'in-progress' | 'completed'
  adminReply?: string
  createdAt: Date
}

export default function ComplaintSystem() {
  const [complaints, setComplaints] = useState<Complaint[]>([])
  const [view, setView] = useState<'user' | 'admin-login' | 'admin'>('user')
  const [password, setPassword] = useState('')
  const [passwordError, setPasswordError] = useState('')
  const [activeTab, setActiveTab] = useState<'submit' | 'check'>('submit')
  const [searchId, setSearchId] = useState('')
  const [foundComplaint, setFoundComplaint] = useState<Complaint | null>(null)
  const [searchError, setSearchError] = useState('')

  // Form states
  const [name, setName] = useState('')
  const [userId, setUserId] = useState('')
  const [description, setDescription] = useState('')
  const [formError, setFormError] = useState('')

  // Admin states
  const [selectedStatus, setSelectedStatus] = useState<'pending' | 'in-progress' | 'completed'>('pending')
  const [adminReply, setAdminReply] = useState('')
  const [selectedComplaint, setSelectedComplaint] = useState<Complaint | null>(null)

  const generateId = () => {
    // Generate 8-digit numeric ID
    return Math.floor(10000000 + Math.random() * 90000000).toString()
  }

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    if (!name || !userId || !description) {
      setFormError('All fields are required')
      return
    }

    const newComplaint: Complaint = {
      id: generateId(),
      name,
      userId,
      description,
      status: 'pending',
      createdAt: new Date()
    }

    setComplaints([...complaints, newComplaint])
    setName('')
    setUserId('')
    setDescription('')
    setFormError('')
    setActiveTab('check')
    setFoundComplaint(newComplaint)
  }

  const handleSearch = (e: React.FormEvent) => {
    e.preventDefault()
    if (!searchId) {
      setSearchError('Please enter a complaint ID')
      return
    }

    // Validate numeric input
    if (!/^\d+$/.test(searchId)) {
      setSearchError('Complaint ID must contain only numbers')
      return
    }

    const complaint = complaints.find(c => c.id === searchId)
    if (!complaint) {
      setSearchError('No complaint found with that ID')
      setFoundComplaint(null)
      return
    }

    setFoundComplaint(complaint)
    setSearchError('')
  }

  const handleAdminLogin = (e: React.FormEvent) => {
    e.preventDefault()
    if (password !== 'LEHBITEAM') {
      setPasswordError('Incorrect password')
      return
    }
    setPasswordError('')
    setView('admin')
  }

  const updateComplaintStatus = (complaintId: string, status: 'pending' | 'in-progress' | 'completed') => {
    setComplaints(complaints.map(c => 
      c.id === complaintId ? { ...c, status } : c
    ))
    if (foundComplaint && foundComplaint.id === complaintId) {
      setFoundComplaint({ ...foundComplaint, status })
    }
  }

  const addAdminReply = (complaintId: string, reply: string) => {
    setComplaints(complaints.map(c => 
      c.id === complaintId ? { ...c, adminReply: reply } : c
    ))
    if (foundComplaint && foundComplaint.id === complaintId) {
      setFoundComplaint({ ...foundComplaint, adminReply: reply })
    }
    setAdminReply('')
    setSelectedComplaint(null)
  }

  return (
    <div className="min-h-screen bg-gray-50">
      <header className="bg-primary text-primary-foreground py-4 shadow-md">
        <div className="container mx-auto px-4">
          <h1 className="text-2xl font-bold flex items-center gap-2">
            {view === 'admin' ? (
              <button onClick={() => setView('user')} className="flex items-center gap-1">
                <ArrowLeft className="h-5 w-5" />
                <Shield className="h-5 w-5" />
              </button>
            ) : view === 'admin-login' ? (
              <button onClick={() => setView('user')} className="flex items-center gap-1">
                <ArrowLeft className="h-5 w-5" />
                <Shield className="h-5 w-5" />
              </button>
            ) : (
              <User className="h-5 w-5" />
            )}
            Complaint Management System
          </h1>
        </div>
      </header>

      <main className="container mx-auto px-4 py-8">
        {view === 'user' && (
          <div className="max-w-2xl mx-auto">
            <Card>
              <CardHeader>
                <div className="flex border-b">
                  <button
                    className={`px-4 py-2 font-medium ${activeTab === 'submit' ? 'border-b-2 border-primary text-primary' : 'text-gray-500'}`}
                    onClick={() => setActiveTab('submit')}
                  >
                    Submit Complaint
                  </button>
                  <button
                    className={`px-4 py-2 font-medium ${activeTab === 'check' ? 'border-b-2 border-primary text-primary' : 'text-gray-500'}`}
                    onClick={() => setActiveTab('check')}
                  >
                    Check Status
                  </button>
                </div>
              </CardHeader>
              <CardContent className="pt-6">
                {activeTab === 'submit' ? (
                  <form onSubmit={handleSubmit}>
                    <div className="space-y-4">
                      <div>
                        <Label htmlFor="name">Name</Label>
                        <Input
                          id="name"
                          value={name}
                          onChange={(e) => setName(e.target.value)}
                          placeholder="Your name"
                        />
                      </div>
                      <div>
                        <Label htmlFor="userId">ID Number</Label>
                        <Input
                          id="userId"
                          value={userId}
                          onChange={(e) => setUserId(e.target.value)}
                          placeholder="Your ID number"
                        />
                      </div>
                      <div>
                        <Label htmlFor="description">Complaint Description</Label>
                        <Textarea
                          id="description"
                          value={description}
                          onChange={(e) => setDescription(e.target.value)}
                          placeholder="Describe your complaint in detail"
                          rows={5}
                        />
                      </div>
                      {formError && <p className="text-red-500 text-sm">{formError}</p>}
                      <Button type="submit" className="w-full">
                        Submit Complaint
                      </Button>
                    </div>
                  </form>
                ) : (
                  <div>
                    <form onSubmit={handleSearch} className="mb-6">
                      <div className="flex gap-2">
                        <Input
                          value={searchId}
                          onChange={(e) => setSearchId(e.target.value)}
                          placeholder="Enter complaint number"
                          inputMode="numeric"
                        />
                        <Button type="submit">
                          <Search className="h-4 w-4 mr-2" />
                          Search
                        </Button>
                      </div>
                      {searchError && <p className="text-red-500 text-sm mt-2">{searchError}</p>}
                    </form>

                    {foundComplaint && (
                      <div className="space-y-4">
                        <div className="flex items-center justify-between bg-gray-100 p-3 rounded-md">
                          <div className="flex items-center gap-2">
                            <div className={`h-3 w-3 rounded-full ${
                              foundComplaint.status === 'pending' ? 'bg-yellow-500' :
                              foundComplaint.status === 'in-progress' ? 'bg-blue-500' :
                              'bg-green-500'
                            }`} />
                            <span className="font-medium capitalize">
                              Status: {foundComplaint.status.replace('-', ' ')}
                            </span>
                          </div>
                          <span className="text-sm text-gray-600">
                            Complaint #{foundComplaint.id}
                          </span>
                        </div>

                        <Card>
                          <CardHeader>
                            <CardTitle>Complaint Details</CardTitle>
                            <CardDescription>
                              Submitted on {foundComplaint.createdAt.toLocaleDateString()}
                            </CardDescription>
                          </CardHeader>
                          <CardContent className="space-y-4">
                            <div>
                              <p className="font-medium">Name:</p>
                              <p className="mt-1 text-gray-700">{foundComplaint.name}</p>
                            </div>
                            <div>
                              <p className="font-medium">ID Number:</p>
                              <p className="mt-1 text-gray-700">{foundComplaint.userId}</p>
                            </div>
                            <div>
                              <p className="font-medium">Description:</p>
                              <p className="mt-1 text-gray-700">{foundComplaint.description}</p>
                            </div>
                            {foundComplaint.adminReply && (
                              <div className="bg-blue-50 p-4 rounded-md">
                                <p className="font-medium text-blue-800">Admin Response:</p>
                                <p className="mt-1 text-blue-700">{foundComplaint.adminReply}</p>
                              </div>
                            )}
                          </CardContent>
                        </Card>
                      </div>
                    )}
                  </div>
                )}
              </CardContent>
            </Card>

            <div className="mt-4 text-center">
              <Button
                variant="outline"
                onClick={() => setView('admin-login')}
                className="flex items-center gap-2 mx-auto"
              >
                <Shield className="h-4 w-4" />
                Admin Login
              </Button>
            </div>
          </div>
        )}

        {view === 'admin-login' && (
          <div className="max-w-md mx-auto">
            <Card>
              <CardHeader>
                <CardTitle className="flex items-center gap-2">
                  <Shield className="h-5 w-5" />
                  Admin Login
                </CardTitle>
                <CardDescription>
                  Enter the admin password to access the complaint management dashboard
                </CardDescription>
              </CardHeader>
              <CardContent>
                <form onSubmit={handleAdminLogin}>
                  <div className="space-y-4">
                    <div>
                      <Label htmlFor="password">Password</Label>
                      <Input
                        id="password"
                        type="password"
                        value={password}
                        onChange={(e) => setPassword(e.target.value)}
                        placeholder="Enter admin password"
                      />
                      {passwordError && <p className="text-red-500 text-sm">{passwordError}</p>}
                    </div>
                    <Button type="submit" className="w-full">
                      Login
                    </Button>
                  </div>
                </form>
              </CardContent>
            </Card>
          </div>
        )}

        {view === 'admin' && (
          <div>
            <h2 className="text-xl font-bold mb-6 flex items-center gap-2">
              <Shield className="h-5 w-5" />
              Complaint Dashboard
            </h2>

            <div className="grid gap-6">
              {complaints.length === 0 ? (
                <Card>
                  <CardContent className="py-8 text-center">
                    <p className="text-gray-500">No complaints have been submitted yet</p>
                  </CardContent>
                </Card>
              ) : (
                complaints.map(complaint => (
                  <Card key={complaint.id}>
                    <CardHeader>
                      <div className="flex justify-between items-start">
                        <div>
                          <CardTitle>Complaint #{complaint.id}</CardTitle>
                          <CardDescription>
                            From: {complaint.name} (ID: {complaint.userId})
                          </CardDescription>
                        </div>
                        <div className="flex items-center gap-2">
                          <div className={`h-2 w-2 rounded-full ${
                            complaint.status === 'pending' ? 'bg-yellow-500' :
                            complaint.status === 'in-progress' ? 'bg-blue-500' :
                            'bg-green-500'
                          }`} />
                          <span className="text-sm capitalize">
                            {complaint.status.replace('-', ' ')}
                          </span>
                        </div>
                      </div>
                    </CardHeader>
                    <CardContent>
                      <p className="mb-4">{complaint.description}</p>
                      
                      {selectedComplaint?.id === complaint.id ? (
                        <div className="space-y-4">
                          <div>
                            <Label>Update Status</Label>
                            <Select
                              value={selectedStatus}
                              onValueChange={(value: 'pending' | 'in-progress' | 'completed') => {
                                setSelectedStatus(value)
                                updateComplaintStatus(complaint.id, value)
                              }}
                            >
                              <SelectTrigger className="w-[180px]">
                                <SelectValue placeholder="Select status" />
                              </SelectTrigger>
                              <SelectContent>
                                <SelectItem value="pending">Pending</SelectItem>
                                <SelectItem value="in-progress">In Progress</SelectItem>
                                <SelectItem value="completed">Completed</SelectItem>
                              </SelectContent>
                            </Select>
                          </div>
                          <div>
                            <Label htmlFor="reply">Admin Reply</Label>
                            <Textarea
                              id="reply"
                              value={adminReply}
                              onChange={(e) => setAdminReply(e.target.value)}
                              placeholder="Enter your response to the user"
                              rows={3}
                            />
                          </div>
                          <div className="flex gap-2">
                            <Button
                              onClick={() => addAdminReply(complaint.id, adminReply)}
                              disabled={!adminReply}
                            >
                              Submit Reply
                            </Button>
                            <Button
                              variant="outline"
                              onClick={() => setSelectedComplaint(null)}
                            >
                              Cancel
                            </Button>
                          </div>
                        </div>
                      ) : (
                        <div className="flex gap-2">
                          <Button
                            onClick={() => {
                              setSelectedComplaint(complaint)
                              setSelectedStatus(complaint.status)
                              setAdminReply(complaint.adminReply || '')
                            }}
                          >
                            {complaint.adminReply ? 'Update Response' : 'Respond'}
                          </Button>
                          {complaint.adminReply && (
                            <div className="bg-blue-50 p-3 rounded-md flex-1">
                              <p className="font-medium text-blue-800">Your Response:</p>
                              <p className="text-blue-700">{complaint.adminReply}</p>
                            </div>
                          )}
                        </div>
                      )}
                    </CardContent>
                    <CardFooter>
                      <p className="text-sm text-gray-500">
                        Submitted on {complaint.createdAt.toLocaleDateString()}
                      </p>
                    </CardFooter>
                  </Card>
                ))
              )}
            </div>
          </div>
        )}
      </main>
    </div>
  )
}
