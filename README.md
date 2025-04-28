// Setting up the basic Next.js app structure for Retreat Haven

// --- Package.json ---
{
  "name": "retreat-haven",
  "version": "1.0.0",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "14.0.4",
    "react": "18.2.0",
    "react-dom": "18.2.0"
  },
  "devDependencies": {
    "typescript": "5.2.2"
  }
}

// --- /pages/index.tsx ---
import Link from 'next/link';

export default function Home() {
  return (
    <main className="min-h-screen flex flex-col items-center justify-center bg-gradient-to-b from-green-200 to-blue-100">
      <h1 className="text-5xl font-bold text-gray-800 mb-6">
        Welcome to Retreat Haven
      </h1>
      <p className="text-lg text-gray-600 mb-8">
        Find and book exclusive wellness retreats.
      </p>
      <Link href="/browse">
        <button className="px-6 py-3 bg-green-600 text-white rounded-full hover:bg-green-700">
          Browse Retreats
        </button>
      </Link>
    </main>
  );
}

// --- /pages/browse.tsx ---
import Link from 'next/link';
import { useEffect, useState } from 'react';

interface Retreat {
  id: number;
  title: string;
  location: string;
  date: string;
}

export default function Browse() {
  const [retreats, setRetreats] = useState<Retreat[]>([]);

  useEffect(() => {
    async function fetchRetreats() {
      const res = await fetch('/api/retreats');
      const data = await res.json();
      setRetreats(data);
    }
    fetchRetreats();
  }, []);

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">Available Retreats</h1>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        {retreats.map((retreat) => (
          <div key={retreat.id} className="p-6 bg-white rounded-xl shadow-md">
            <h2 className="text-2xl font-semibold">{retreat.title}</h2>
            <p className="text-gray-600">{retreat.location}</p>
            <p className="text-gray-500">{retreat.date}</p>
            <Link href={`/retreat/${retreat.id}`}>
              <button className="mt-4 px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700">
                View Retreat
              </button>
            </Link>
          </div>
        ))}
      </div>
    </div>
  );
}

// --- /pages/host.tsx ---
import { useState } from 'react';

export default function Host() {
  const [form, setForm] = useState({ title: '', location: '', date: '', description: '' });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const res = await fetch('/api/retreats', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form),
    });
    if (res.ok) {
      alert('Retreat created successfully!');
      setForm({ title: '', location: '', date: '', description: '' });
    }
  };

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">Host Your Retreat</h1>
      <form onSubmit={handleSubmit} className="space-y-4">
        <input className="w-full p-2 border rounded" name="title" value={form.title} onChange={handleChange} placeholder="Retreat Title" required />
        <input className="w-full p-2 border rounded" name="location" value={form.location} onChange={handleChange} placeholder="Location" required />
        <input className="w-full p-2 border rounded" name="date" value={form.date} onChange={handleChange} placeholder="Available Dates" required />
        <textarea className="w-full p-2 border rounded" name="description" value={form.description} onChange={handleChange} placeholder="Retreat Description" rows={4} required />
        <button type="submit" className="px-6 py-2 bg-green-600 text-white rounded hover:bg-green-700">Submit Retreat</button>
      </form>
    </div>
  );
}

// --- /pages/api/retreats/index.ts ---
import { NextApiRequest, NextApiResponse } from 'next';

let retreats: any[] = [
  { id: 1, title: "Yoga in Bali", location: "Bali, Indonesia", date: "July 10-17" },
  { id: 2, title: "Mindfulness Retreat", location: "Sedona, Arizona", date: "September 5-12" },
];

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'GET') {
    res.status(200).json(retreats);
  } else if (req.method === 'POST') {
    const { title, location, date, description } = req.body;
    const newRetreat = {
      id: retreats.length + 1,
      title,
      location,
      date,
      description,
    };
    retreats.push(newRetreat);
    res.status(201).json(newRetreat);
  } else {
    res.status(405).end();
  }
}
