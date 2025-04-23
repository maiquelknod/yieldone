import { useEffect, useState } from "react";
import { ethers } from "ethers";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Progress } from "@/components/ui/progress";

const CONTRACT_ADDRESS = "0x4b1eE43a451c2Bb430f9B02881174187e6A86Eb2";
const USDT_ADDRESS = "0xc2132d05d31c914a87c6611c10748aeb04b58e8f";
const ABI = [
  // Minimal ABI for interaction (to be fully expanded in real deploy)
  "function invest(uint256 _amount, address _referrer) external",
  "function withdraw() external",
  "function getUserInfo(address _user) external view returns (uint256, uint256, uint256)"
];

export default function YieldOneDashboard() {
  const [account, setAccount] = useState(null);
  const [provider, setProvider] = useState(null);
  const [contract, setContract] = useState(null);
  const [balance, setBalance] = useState(0);
  const [profit, setProfit] = useState(0);
  const [dailyProfit, setDailyProfit] = useState(0);
  const [progressPercent, setProgressPercent] = useState(0);

  useEffect(() => {
    const load = async () => {
      if (typeof window.ethereum !== "undefined") {
        const prov = new ethers.providers.Web3Provider(window.ethereum);
        const signer = prov.getSigner();
        const address = await signer.getAddress();
        const cont = new ethers.Contract(CONTRACT_ADDRESS, ABI, signer);

        const [invested, withdrawn, bonus] = await cont.getUserInfo(address);
        const profitVal = withdrawn.toString() || 0;

        setAccount(address);
        setProvider(prov);
        setContract(cont);
        setBalance(Number(invested) / 1e6);
        setProfit(Number(withdrawn) / 1e6);
        setDailyProfit((Number(invested) * 0.01) / 1e6);
        setProgressPercent((Number(withdrawn) / (Number(invested) * 2)) * 100);
      }
    };
    load();
  }, []);

  const handleInvest = async () => {
    if (!contract || !account) return;
    const usdt = new ethers.Contract(
      USDT_ADDRESS,
      ["function approve(address spender, uint256 amount) external returns (bool)"],
      provider.getSigner()
    );
    await usdt.approve(CONTRACT_ADDRESS, ethers.utils.parseUnits("20", 6));
    await contract.invest(ethers.utils.parseUnits("20", 6), account);
  };

  const handleWithdraw = async () => {
    if (contract) {
      await contract.withdraw();
    }
  };

  return (
    <div className="min-h-screen bg-black text-white flex items-center justify-center p-4">
      <Card className="w-full max-w-md bg-zinc-900 shadow-xl rounded-2xl">
        <CardContent className="p-6">
          <h1 className="text-2xl font-bold text-center mb-6">YieldOne</h1>
          <Progress value={progressPercent} className="h-3 mb-6" />

          <div className="grid grid-cols-2 gap-4 mb-6">
            <div>
              <p className="text-sm text-zinc-400">Saldo atual</p>
              <p className="text-lg font-semibold">{balance} USDT</p>
            </div>
            <div>
              <p className="text-sm text-zinc-400">Lucro acumulado</p>
              <p className="text-lg font-semibold">{profit} USDT</p>
            </div>
            <div>
              <p className="text-sm text-zinc-400">Lucro diário</p>
              <p className="text-lg font-semibold">{dailyProfit.toFixed(2)} USDT</p>
            </div>
          </div>

          <div className="flex gap-4 mb-4">
            <Button onClick={handleInvest} className="w-full bg-green-600 hover:bg-green-700">
              Investir 20 USDT
            </Button>
            <Button onClick={handleWithdraw} className="w-full bg-blue-600 hover:bg-blue-700">
              Sacar
            </Button>
          </div>

          <p className="text-sm text-center text-zinc-400 mt-4">
            Carteira conectada: {account || "Conecte sua MetaMask"}
          </p>
        </CardContent>
      </Card>
    </div>
  );
}
